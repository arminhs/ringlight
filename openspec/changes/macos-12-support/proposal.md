# Proposal

## Why

RingLight is pitched as Edge Light for Macs that can't run macOS Tahoe, but the app targets macOS 14.6, so it won't open on Monterey or Ventura. Even with the target lowered, Monterey users couldn't build it themselves, because the project needs Xcode 16 or later and the last Xcode for macOS 12 is 14.2. They need a ready-made download.

## What Changes

- Lower the app's minimum macOS version from 14.6 to 12.0 (Monterey). This is the lowest version possible: the ring's mouse-avoidance mask needs macOS 12 APIs, and current Xcode (27) cannot target anything older.
- Fix what breaks on Monterey that the compiler can't catch. The Temperature control's `thermometer.medium` icon doesn't exist before macOS 13, so it would be blank on macOS 12; use `thermometer` there.
- Add a GitHub Actions workflow that builds a universal (Apple silicon + Intel), ad-hoc-signed `ringlight.app`. It checks that the build declares macOS 12.0 as its minimum and contains both architectures. When a `vX.Y.Z` tag is pushed, it publishes the zipped app to GitHub Releases.
- Run the same build, without publishing, on pull requests and pushes to `main`, so code that needs a newer macOS fails before a release is tagged.
- Update the README with system requirements, download instructions, and how to open an app that isn't notarized the first time. Keep build-from-source instructions for Macs that can run Xcode 16 or later.

## Capabilities

### New Capabilities

- `platform-support`: which macOS versions and Macs RingLight runs on (macOS 12 and later, Apple silicon and Intel), and that every feature works the same on all of them.
- `release-builds`: how downloadable builds are produced, verified and published on GitHub Releases, and what the README tells users about installing them.

### Modified Capabilities

None. Existing feature behavior is unchanged. `platform-support` requires that it stays the same on macOS 12.

## Impact

- Code: `ringlight.xcodeproj/project.pbxproj`, where the project-level `MACOSX_DEPLOYMENT_TARGET` goes from 14.6 to 12.0. The unit-test target keeps 14.6 because Swift Testing needs macOS 14. In `ringlight/ringlightApp.swift`, `TemperatureSlider` falls back to the `thermometer` icon on macOS 12.
- Info.plist: the generated `LSMinimumSystemVersion` changes from 14.6 to 12.0. There are no entitlement changes and no new macOS permission prompts.
- New file: `.github/workflows/build.yml`. It uses only GitHub's own actions, and only the release job gets write access to repository contents.
- Distribution: release downloads are ad-hoc signed and not notarized, so Gatekeeper warns on first launch. Because camera permission is tied to the app's signature, macOS may ask for camera access again after each update. Developer ID signing and notarization are left for a later change.
- Testing: GitHub no longer provides macOS 12 runners, so the workflow can only check the build's metadata. Behavior on Monterey must be checked by hand on a macOS 12 Mac or virtual machine.
- Docs: `README.md`, plus the project context in `openspec/config.yaml` (deployment target, release workflow).
