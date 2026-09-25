# Proposal

## Why

The camera can keep running after the control panel closes. It starts on a background queue, and the app only records the session once startup has finished. If the panel closes during startup, nothing is stopped, and the camera (with its in-use indicator) stays on until the panel is opened and closed again. This breaks the camera-preview requirement "Camera runs only while the panel is open". It is also a privacy problem for an app that people keep running during video calls.

## What Changes

- Closing the control panel always stops the camera, even while it is still starting. If startup is in progress, the camera stops as soon as startup finishes, instead of staying on until the next open and close.
- Opening and closing the panel quickly no longer creates a second camera session that the app loses track of. Each opening of the panel owns exactly one session, and that session is stopped when the panel closes.
- Camera start and stop requests run in the order they were made, off the main thread. As a side effect, closing the panel no longer blocks the UI while the camera shuts down.
- The rest of the camera preview works as before: the default camera, the mirrored preview, the permission prompt, and a black preview when no camera is available or access is denied.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `camera-preview`: "Camera runs only while the panel is open" gets new scenarios for closing the panel while the camera is still starting, and for opening and closing it quickly. The requirement's intent does not change. The scenarios spell out the timing that the current implementation gets wrong, so the regression stays covered.

## Impact

- Code: `ringlight/ringlightApp.swift`, mainly `AppDelegate.startCamera()` and `AppDelegate.stopCamera()`, plus one new property for a dedicated camera queue. The popover delegate callbacks and `CameraPreviewView` keep their current behavior.
- Permissions: no new macOS permission prompts and no entitlement or Info.plist changes. The existing camera prompt and `NSCameraUsageDescription` are unchanged.
- Dependencies: none added. Only public AVFoundation APIs are used.
- Docs: no README change, because the README does not describe the camera preview.
