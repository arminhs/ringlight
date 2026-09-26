# Design

## Context

See proposal.md (Why) for the user-facing problem and `specs/camera-preview/spec.md` for the required behavior.

Current flow in `ringlight/ringlightApp.swift`:

- `popoverWillShow` sets `showCameraPreview = true` and calls `startCamera()`. That function builds an `AVCaptureSession` for the default camera and calls `startRunning()` on a global concurrent queue. It assigns `captureSession` on the main queue only after `startRunning()` returns.
- `popoverDidClose` sets `showCameraPreview = false` and calls `stopCamera()`, which calls `captureSession?.stopRunning()` on the main thread and then clears the property.

This leads to two failures:

1. **Close during startup.** `captureSession` is still `nil`, so the stop does nothing. The late assignment then leaves a running session while the panel is closed. The camera stays on until the panel is opened and closed again.
2. **Reopen during startup.** The `captureSession == nil` guard passes again, so a second session is created. The last assignment wins, and the app never stops the other session. If the preview had already attached to the first session, it keeps showing that one.

Constraints:

- `startRunning()` and `stopRunning()` block. Apple's guidance, and its AVCam sample, is to call both on one serial queue off the main thread.
- There is no automated coverage for camera behavior. The fix must be built in Xcode on a Mac and checked by hand.

## Goals / Non-Goals

**Goals:**

- A stop requested when the panel closes always takes effect, even if it arrives before startup finishes.
- The app always holds a reference to the session it must stop, and at most one app-owned session exists per opening of the panel.
- No blocking camera calls run on the main thread.
- Changes stay inside `AppDelegate`'s camera code.

**Non-Goals:**

- Reworking `CameraPreviewView`. It sets the preview layer's frame only in `updateNSView`, and it gets its size right because mouse tracking re-renders SwiftUI on every mouse move. That weakness is older than this bug and is left for a separate change.
- Changing the permission flow (there is no explicit `requestAccess`), camera selection (still the default device when the panel opens), or how the preview looks.
- Keeping one long-lived capture session across openings of the panel.

## Decisions

### 1. One serial queue for every start and stop

`AppDelegate` gets a private serial `DispatchQueue` (label `om.ringlight.camera-session`). `startCamera()` queues `startRunning()` on it, and `stopCamera()` queues `stopRunning()`. Because the queue runs work in order, a stop requested during startup runs right after that start finishes.

Alternatives considered:

- **Keep the global queue and re-check after startup.** After `startRunning()` returns, check on the main queue whether the panel is still open (or compare a generation counter), and stop the session if it isn't. This works, but it spreads the state across two threads and a token. Reopening during startup would still need its own handling. Rejected as easier to get wrong.
- **Assign the session earlier but keep calling `stopRunning()` on the main thread.** This still blocks the UI. It can also call `stopRunning()` while `startRunning()` is running on another thread, which AVFoundation does not promise to handle. Rejected.
- **Wrap the session in a Swift actor.** This adds async code to synchronous AppKit delegate callbacks for only two calls, without any extra guarantee over a serial queue. Rejected.

### 2. Own the session on the main thread from the moment it is created

`startCamera()` builds and configures the session, assigns `captureSession` right away on the main thread, and only then queues the start. `stopCamera()` takes the current session, sets `captureSession = nil` immediately, and queues the stop for that same session:

```swift
func startCamera() {
    guard captureSession == nil else { return }
    // look up device + input first (Decision 3), then configure `session`
    captureSession = session
    cameraQueue.async { session.startRunning() }
}

func stopCamera() {
    guard let session = captureSession else { return }
    captureSession = nil
    cameraQueue.async { session.stopRunning() }
}
```

As a result, `captureSession == nil` reliably means "this opening of the panel has no session". Opening, closing and reopening quickly gives this order on the queue: start S1, stop S1, start S2. Each session is stopped by the close that follows its open, and two app sessions never run at once.

`popoverWillShow` sets `showCameraPreview = true` and then calls `startCamera()` in the same main-thread turn, so the preview already has the session on its first render. `captureSession` therefore does not need to become `@Published`.

Alternative considered: keep one session for the app's lifetime and only start and stop it. That would skip reconfiguring on each open, but it would lock in the camera chosen the first time. Today each opening picks up the current default camera, for example after a webcam is plugged in. Rejected to keep that behavior.

### 3. Configure only after a camera input is available

Look up the default video device and create its input before calling `beginConfiguration()`. If either step fails, leave `captureSession` as `nil` and queue nothing, so the preview stays black as the spec requires. This also removes today's early return, which leaves `beginConfiguration()` without a matching `commitConfiguration()`.

### No private APIs

The change uses only public AVFoundation and Dispatch APIs. It adds no private or undocumented macOS API, so no fallback is needed.

## Risks / Trade-offs

- [A quick open and close still turns the camera on briefly, because the stop waits for the start that is already running] → Accepted. AVFoundation cannot cancel a `startRunning()` in progress. The camera and its indicator turn off as soon as startup finishes, which is what the spec requires.
- [Rapid toggling queues several start/stop pairs, so the camera can take a moment to settle] → The final state always matches the last action. When the panel ends closed, the last queued call is a stop.
- [The preview layer now attaches on the first render, when the preview view may not have its final size yet] → No worse than today: `updateNSView` refreshes the layer's frame on every mouse-move re-render. Making the layer resize with its view is a follow-up (see Non-Goals).
- [The change cannot be built or run in a Linux environment] → Build it in Xcode on a Mac and run the manual checks in tasks.md.
