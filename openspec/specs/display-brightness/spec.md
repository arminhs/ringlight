# display-brightness Specification

## Purpose
Ties the Brightness setting to the hardware brightness of the Mac's main display, so one slider controls both how bright the ring is and how bright the screen is.

## Requirements

### Requirement: Start from the current display brightness
On launch, the system SHALL read the current brightness of the main display and use it as the initial Brightness setting, leaving the display's brightness unchanged. If the brightness cannot be read, Brightness SHALL start at 100%.

#### Scenario: Launch with a dimmed display
- **WHEN** the main display is at 60% brightness and the app launches
- **THEN** the Brightness slider shows 60%
- **AND** the display stays at 60% brightness

#### Scenario: Brightness cannot be read
- **WHEN** the app launches and the main display's brightness cannot be read
- **THEN** the Brightness slider shows 100%

### Requirement: Brightness slider sets display brightness
When the user moves the Brightness slider (10% to 100%), the system SHALL set the main display's hardware brightness to the same level, in addition to changing the ring's intensity.

#### Scenario: Lower the brightness
- **WHEN** the user moves the Brightness slider to 40%
- **THEN** the main display's brightness changes to 40%
- **AND** the ring is drawn at 40% opacity

### Requirement: Graceful fallback
If the system cannot control the display's brightness (for example on a display without brightness control, or when the macOS interface for it is unavailable), the Brightness slider SHALL still adjust the ring, and the app SHALL keep working without showing an error.

#### Scenario: Display without brightness control
- **WHEN** the main display does not support brightness control and the user moves the Brightness slider
- **THEN** only the ring's intensity changes
- **AND** no error is shown
