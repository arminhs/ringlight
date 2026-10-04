# Spec Delta

## MODIFIED Requirements

### Requirement: Camera runs only while the panel is open
The system SHALL stop the camera when the control panel closes, including when the panel closes before the camera has finished starting, and SHALL NOT use the camera while the panel is closed.

#### Scenario: Close the panel
- **WHEN** the user closes the control panel
- **THEN** the camera stops and the camera-in-use indicator turns off

#### Scenario: Close the panel while the camera is starting
- **WHEN** the user opens the control panel and closes it again before live video appears in the preview
- **THEN** the camera stops as soon as it has finished starting, without the panel being opened again
- **AND** the camera-in-use indicator turns off

#### Scenario: Open and close the panel rapidly
- **WHEN** the user opens and closes the control panel several times in quick succession, ending with the panel closed
- **THEN** the camera stops
- **AND** the camera-in-use indicator turns off and stays off
