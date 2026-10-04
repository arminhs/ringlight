# camera-preview Specification

## Purpose
Shows a live, mirrored preview from the Mac's camera in the control panel so the user can check how they are lit while adjusting the ring light.

## Requirements

### Requirement: Live preview while the panel is open
When the control panel opens, the system SHALL start the Mac's default camera and show its live video at the top of the panel, 140 points tall with rounded corners, filling the frame by cropping rather than letterboxing.

#### Scenario: Open the panel
- **WHEN** camera access is granted and the user opens the control panel
- **THEN** a live camera preview appears at the top of the panel

### Requirement: Mirrored image
The system SHALL mirror the preview horizontally, like looking in a mirror.

#### Scenario: Raise the right hand
- **WHEN** the user raises their right hand in front of the camera
- **THEN** the hand appears on the right side of the preview

### Requirement: Camera runs only while the panel is open
The system SHALL stop the camera when the control panel closes and SHALL NOT use the camera while the panel is closed.

#### Scenario: Close the panel
- **WHEN** the user closes the control panel
- **THEN** the camera stops and the camera-in-use indicator turns off

### Requirement: Camera permission and unavailability
Camera use SHALL go through the macOS camera permission prompt, which explains that the camera shows a preview in the settings menu. If no camera is available or access is denied, the preview area SHALL stay black and all other controls SHALL keep working.

#### Scenario: Camera access denied
- **WHEN** the user has denied camera access and opens the control panel
- **THEN** the preview area is black
- **AND** the ring light controls still work

#### Scenario: No camera connected
- **WHEN** the Mac has no camera and the user opens the control panel
- **THEN** the preview area is black
- **AND** the ring light controls still work
