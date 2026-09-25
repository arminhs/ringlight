# Design

## Context

See proposal.md (Why) for motivation. The requirements are in `specs/platform-support/spec.md` and `specs/release-builds/spec.md`.

Current project state:

- `MACOSX_DEPLOYMENT_TARGET = 14.6` is set at the project level (Debug and Release). The app target and the UI-test target inherit it, and `ringlightTests` sets 14.6 explicitly. The Info.plist is generated, so `LSMinimumSystemVersion` follows the deployment target.
- Local builds use "Sign to Run Locally": there is no `DEVELOPMENT_TEAM`, no hardened runtime, and `CODE_SIGN_STYLE = Automatic`.
- The repo has no shared Xcode scheme (no `xcshareddata`) and no `.github/` directory.
- The project uses Xcode 16's file format (`objectVersion = 77`), so it opens only in Xcode 16 or later, which needs macOS 14.5 or later.
- Release builds already target `$(ARCHS_STANDARD)` (arm64 and x86_64); only Debug sets `ONLY_ACTIVE_ARCH = YES`.

API audit against macOS 12, based on Apple's documentation data:

- These APIs need macOS 12 and set the floor: `Canvas`, `GraphicsContext.addFilter` and `GraphicsContext.blendMode` (all used by the mouse-avoidance mask), and `Color(nsColor:)`.
- Everything else is older and fine: `Settings`, `NSApplicationDelegateAdaptor` and `ignoresSafeArea` (macOS 11); the `.switch` and `.checkbox` toggle styles, the `.plain` button style, `LinearGradient(colors:…)`, `controlSize` and `#Preview` (macOS 10.15). The menu bar uses `NSStatusItem` and `NSPopover` rather than `MenuBarExtra` (macOS 13), so nothing changes there.
- `Font.system(size:weight:design:)` has two overloads: the one with optional parameters needs macOS 13, and the original works on 10.15. Swift prefers overloads that are available at the deployment target, so the four call sites should resolve to the 10.15 version.
- The compiler does not check SF Symbol names. `sun.max.fill`, `rectangle.expand.vertical` and `star.fill` date from SF Symbols 1 (macOS 11). `thermometer.medium` is new in SF Symbols 4 (macOS 13), where `thermometer` was renamed to it, so the Temperature icon is blank on macOS 12.
- Brightness control uses the private `DisplayServices` framework through `dlopen`/`dlsym`. This change does not touch that code.

Toolchain facts:

- Xcode 27 supports macOS deployment targets from 12.0 to 27.x.
- GitHub-hosted runners include `macos-26` (arm64, Xcode 26.6 by default), `macos-15` and Intel variants. There are no macOS 12 runners.

## Goals / Non-Goals

**Goals:**

- The app binary and Info.plist declare macOS 12.0, and the app behaves on macOS 12 as it does on current macOS.
- One workflow builds, verifies and, for version tags, publishes a universal, ad-hoc-signed app.
- Every pull request is compiled against the 12.0 target, so the floor does not regress.

**Non-Goals:**

- Developer ID signing and notarization, which need a paid Apple Developer account and CI secrets. The workflow leaves room to add them later (Decision 6).
- Making the project buildable on macOS 12 with Xcode 14.2.
- Running tests in CI or on macOS 12. The tests are placeholders, UI tests need a logged-in GUI session with permissions, and there are no hosted macOS 12 runners.
- Auto-update, DMG packaging or a Homebrew cask.

## Decisions

### 1. Deployment target 12.0 at the project level; unit tests stay at 14.6

Change the project-level `MACOSX_DEPLOYMENT_TARGET` from 14.6 to 12.0 in both Debug and Release. The app and UI-test targets inherit it. `ringlightTests` keeps its explicit 14.6, because Swift Testing's package manifest requires macOS 14, and unit tests only run on the build Mac, which already has macOS 14.5 or later for Xcode 16. A test bundle can have a higher deployment target than its host app.

We chose 12.0 rather than 11 because the mouse-avoidance mask (`Canvas`/`GraphicsContext`) and `Color(nsColor:)` need macOS 12, and Xcode 27 cannot target anything below 12.0. Going lower would mean rewriting the mask, and it would stop building with current Xcode.

Alternative considered: set the value on the app target only. Rejected, because the setting lives at the project level today; changing it there is one line per configuration.

### 2. Choose the thermometer symbol at runtime

`TemperatureSlider` uses `thermometer.medium` on macOS 13 and later, and `thermometer` on macOS 12, selected with `if #available(macOS 13, *)`.

Alternatives considered:

- **Always use `thermometer`.** It is the same glyph, and the old name still resolves on newer systems, but that depends on a deprecated alias. Rejected in favor of an explicit check.
- **Bundle a custom symbol.** Too much for a single icon. Rejected.

### 3. Let the compiler finish the audit

After changing the target, build. Any API newer than macOS 12 then fails to compile. The audit above expects none. If the `Font.system` calls resolve to the macOS 13 overload after all, pass explicit non-optional arguments (for example `weight: .regular`) to select the 10.15 version. From then on, the pull-request build (Decision 5) keeps the code compatible.

### 4. One workflow with two jobs: `build` on macOS, `release` on Linux

`.github/workflows/build.yml` runs on pull requests, on pushes to `main`, on pushed tags matching `v[0-9]+.[0-9]+.[0-9]+`, and on manual `workflow_dispatch`.

The **build** job runs on every trigger. It uses `runs-on: macos-26`, pins Xcode with `DEVELOPER_DIR=/Applications/Xcode_26.6.app/Contents/Developer`, and has `permissions: contents: read`. It has three steps:

- **Build:** `xcodebuild -project ringlight.xcodeproj -target ringlight -configuration Release` with ad-hoc signing (`CODE_SIGN_IDENTITY=-`, `CODE_SIGN_STYLE=Manual`, empty `DEVELOPMENT_TEAM`). `CURRENT_PROJECT_VERSION` is always the run number. On tag builds, `MARKETING_VERSION` is the tag without its "v".
- **Verify:** the job fails if any check fails:
  - `lipo -archs` lists both `arm64` and `x86_64`
  - `LSMinimumSystemVersion` is `12.0`
  - `vtool -show-build` reports `minos 12.0` for both slices
  - `codesign --verify --deep --strict` passes
  - on tag builds, `CFBundleShortVersionString` equals the tag's version
- **Package:** zip the app with `ditto -c -k --keepParent` into `ringlight-<version>.zip`, write a `shasum -a 256` file next to it, and upload both with `actions/upload-artifact` using a short retention period.

The **release** job runs for tags only. It `needs: build`, uses `runs-on: ubuntu-latest` and has `permissions: contents: write`. It downloads the artifact and runs `gh release create <tag> <zip> <sha256> --generate-notes`, adding a one-line install note that links to the README.

Why these choices:

- **`-target` instead of `-scheme` or `archive`.** The repo has no shared scheme, and `xcodebuild archive` requires one. A plain Release build is enough for an ad-hoc-signed app. Committing a shared scheme becomes the natural step when Developer ID export is added.
- **Pinned runner image and Xcode instead of `macos-latest`.** Toolchain upgrades then happen on purpose, in a pull request that bumps the pin, instead of silently. `macos-26` with Xcode 26.6 is the current stable setup; Xcode 27 is still in preview on the runners.
- **Split jobs.** The job that runs `xcodebuild` never holds a write token, and the job with write access only uploads files. There are no third-party actions: `actions/checkout`, `actions/upload-artifact` and `actions/download-artifact` come from GitHub, and publishing uses the preinstalled `gh` CLI with `GITHUB_TOKEN`.
- **Version taken from the tag instead of a manual bump.** Tagging is the only release step. The trade-off is that the project file keeps `1.0`, so local builds don't show the release number.
- **zip made with `ditto` instead of a DMG.** A zip is standard for GitHub downloads, keeps the app's signature intact and needs no extra tooling.

### 5. Pull-request builds guard the macOS 12 floor

Pull requests and pushes to `main` run the same `build` job, without the release job. There are no macOS 12 runners and the tests are placeholders, so compiling against the 12.0 target and checking the build metadata is the practical way to catch regressions.

### 6. Ad-hoc signing now, Developer ID later

Downloads are ad-hoc signed, which gives a valid signature (required to run on Apple silicon), but they are not notarized, so Gatekeeper warns on first launch. The README documents the first-launch steps:

- macOS 12 to 14: Control-click the app, choose Open, then confirm.
- macOS 15 and later: try to open the app, then go to System Settings › Privacy & Security and choose Open Anyway.

Adding Developer ID later means these steps, all in the `build` job before packaging:

1. Import the certificate from secrets.
2. Sign with the hardened runtime.
3. Run `notarytool submit --wait`.
4. Staple the notarization ticket.

### 7. Make the ring's cut-out independent of the fill rule

The ring's path is the outer rounded rect plus the inner one, both drawn in the same direction. It relied on the even-odd fill rule, applied through a `fill` wrapper on `RoundedRingShape`, to leave the middle empty. In the first test of the downloaded build, the ring rendered as a filled rectangle, so the even-odd rule was not in effect there.

The inner rect is now mirrored onto itself with `CGAffineTransform(a: -1, b: 0, c: 0, d: 1, tx: 2 × midX, ty: 0)`. It covers the same area but winds the opposite way, so the middle stays empty under both the non-zero and the even-odd rule. The ring's geometry and look do not change.

Alternatives considered:

- **Find out why the even-odd rule was lost and fix only that.** The cause could be the overload the `fill` wrapper resolves to at the lower deployment target, or the older SwiftUI runtime. Either way, the shape would still break whenever the rule is lost. Rejected.
- **Stroke a rounded rect instead of filling a path.** The inner corner radius would then follow from the stroke instead of the current `max(radius - 0.6 × thickness, 20)`, which changes the look. Rejected.

### Private API

This change adds no private API. The existing `DisplayServices` brightness calls keep their fallback: if the framework or one of its functions is missing on a Mac, brightness control silently does nothing and the slider still adjusts the ring, as the display-brightness spec requires.

## Risks / Trade-offs

- [There is no automated testing on macOS 12, and SwiftUI may render differently there, for example the `Canvas` mask or the popover layout] → Before the first release, run the manual checklist on a macOS 12 Mac or virtual machine using the workflow's build artifact. The camera and brightness need real hardware, because virtual machines don't provide them.
- [`DisplayServices` may behave differently on Intel Macs or older macOS] → The existing silent fallback covers it. Check on an Intel Mac with Monterey if one is available.
- [Ad-hoc signing means Gatekeeper warnings on first launch and camera permission prompts after updates, because camera permission is tied to the code signature and ad-hoc signatures change with every build] → Documented in the README. Developer ID signing is the follow-up that fixes both.
- [GitHub retires runner images and Xcode versions] → The pinned versions then fail loudly; bump them in a pull request.
- [A future Xcode could raise its minimum macOS target above 12, as Xcode 27 raised it to 12] → The pinned Xcode keeps working until its runner image is retired. Revisit the supported range then.
- [`xcodebuild -target` differs slightly from an archive build, for example symbols are not stripped] → Acceptable for an app this small. Switch to `archive` with a shared scheme when Developer ID signing is added.
