# Tasks

## 1. Serialize camera start and stop

- [ ] 1.1 Add a private serial `cameraQueue` (`DispatchQueue(label: "om.ringlight.camera-session")`) to `AppDelegate` in `ringlight/ringlightApp.swift`, and verify that the app target builds in Xcode (⌘B)
- [ ] 1.2 In `startCamera()`, return early when `captureSession` is already set, and look up the default video device and create its input before `beginConfiguration()`, returning without a session if either step fails (design Decision 3); verify that the app builds
- [ ] 1.3 In `startCamera()`, assign `captureSession` as soon as the session is configured, and queue `startRunning()` on `cameraQueue` instead of `DispatchQueue.global` (design Decision 2); verify that the app builds and that opening the panel still shows a live, mirrored preview
- [ ] 1.4 Rewrite `stopCamera()` to take the current session, set `captureSession = nil`, and queue `stopRunning()` for that session on `cameraQueue`; verify that the app builds and that closing the panel after the preview appears turns off the camera-in-use indicator
- [ ] 1.5 Run `grep -n "startRunning\|stopRunning\|DispatchQueue.global" ringlight/ringlightApp.swift` and verify that `startRunning()` and `stopRunning()` are only called inside `cameraQueue.async` blocks, with no camera work left on `DispatchQueue.global`

## 2. Verify in the running app

- [ ] 2.1 Close during startup: click the menu bar icon, then click outside the panel before live video appears; verify that the camera-in-use indicator turns off within a second or two and stays off without reopening the panel (scenario "Close the panel while the camera is starting")
- [ ] 2.2 Rapid toggling: open and close the panel about five times in quick succession with the menu bar icon, ending with it closed; verify that the indicator ends up off and stays off, then open the panel once more and verify that the live, mirrored preview appears (scenario "Open and close the panel rapidly")
- [ ] 2.3 Normal use: open the panel, wait for the preview, then close it; verify that the indicator turns off and the panel closes without the UI stalling (scenario "Close the panel")
- [ ] 2.4 Permissions: run `tccutil reset Camera om.ringlight`, relaunch, open the panel and allow camera access; verify that the preview works and the indicator turns off after closing; then deny access in System Settings › Privacy & Security › Camera, reopen the panel, and verify that the preview stays black while the other controls keep working
