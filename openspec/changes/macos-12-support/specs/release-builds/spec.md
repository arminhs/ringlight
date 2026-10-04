# Spec Delta

## Purpose

Publishes ready-to-run RingLight downloads on GitHub so that people can install it without Xcode, including on macOS 12, where the project cannot be built.

## ADDED Requirements

### Requirement: Tagged versions publish a download
Pushing a version tag of the form `vMAJOR.MINOR.PATCH` SHALL build RingLight in its Release configuration and publish a GitHub Release for that tag, with the zipped `ringlight.app` and its SHA-256 checksum attached. If a release for the tag already exists, for example because it was published in GitHub's web UI, the two files SHALL be attached to that release. The app's version SHALL match the tag without its leading "v".

#### Scenario: Push a version tag
- **WHEN** a maintainer pushes the tag `v1.1.0`
- **THEN** a GitHub Release named `v1.1.0` is published with the zipped app and its checksum
- **AND** the downloaded app shows version 1.1.0 in Finder's Get Info

#### Scenario: Release published in GitHub's web UI
- **WHEN** a maintainer publishes a release for a new tag `v1.1.0` in GitHub's web UI
- **THEN** the zipped app and its checksum are attached to that release
- **AND** the release keeps the title and notes the maintainer wrote

#### Scenario: Tag that is not a version
- **WHEN** a maintainer pushes a tag that does not match `vMAJOR.MINOR.PATCH`
- **THEN** no release is built or published

### Requirement: Verified universal build for macOS 12
Before anything is published, the build SHALL be checked to contain native code for both Apple silicon (arm64) and Intel (x86_64), to declare macOS 12.0 as its minimum system version, and to carry a valid code signature. If any check fails, the workflow SHALL fail and SHALL NOT publish a release.

#### Scenario: Checks pass
- **WHEN** a tagged build contains arm64 and x86_64 code, declares macOS 12.0 as its minimum, and has a valid signature
- **THEN** the release is published

#### Scenario: Wrong minimum version
- **WHEN** a tagged build declares a minimum macOS version other than 12.0
- **THEN** the workflow fails and no release is published

#### Scenario: Missing architecture
- **WHEN** a tagged build lacks either arm64 or x86_64 code
- **THEN** the workflow fails and no release is published

### Requirement: Build check on pull requests
Pull requests and pushes to `main` SHALL run the same build and checks as a release, and SHALL NOT publish anything.

#### Scenario: Pull request uses an API that needs a newer macOS
- **WHEN** a pull request adds code that calls an API unavailable on macOS 12 without an availability check
- **THEN** the build check fails

#### Scenario: Passing pull request
- **WHEN** a pull request builds and passes the checks
- **THEN** no GitHub Release is created

### Requirement: Install instructions for downloads that are not notarized
Release builds are ad-hoc signed and not notarized. The README SHALL link to the latest release and explain how to install the app and open it for the first time: on macOS 12 to 14 by Control-clicking the app and choosing Open, and on macOS 15 and later by allowing it in System Settings › Privacy & Security.

#### Scenario: First launch on macOS 12
- **WHEN** a user on macOS 12 downloads the latest release and opens it for the first time
- **THEN** following the README's steps gets them past the Gatekeeper warning and the app starts

#### Scenario: First launch on macOS 15 or later
- **WHEN** a user on macOS 15 or later sees the Gatekeeper warning for the downloaded app
- **THEN** the README tells them where in System Settings to allow it
