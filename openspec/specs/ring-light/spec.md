# ring-light Specification

## Purpose
Draws a glowing, rectangular ring of light around the edges of the screen so the user's face is lit during video calls, without getting in the way of normal work.

## Requirements

### Requirement: Screen-edge ring
The system SHALL draw a rectangular ring of light with rounded corners that follows the edges of the screen. The ring's outer edge SHALL be inset 20 points from the left, right and bottom edges of the screen and 20 points below the menu bar, with a 40-point corner radius.

#### Scenario: Ring shown on launch
- **WHEN** the app launches
- **THEN** a ring of light is drawn around the edges of the screen
- **AND** the ring stays clear of the menu bar

### Requirement: Main display only
The system SHALL draw the ring on a single display: the main display at the time the app launches.

#### Scenario: Several displays connected
- **WHEN** the app launches while more than one display is connected
- **THEN** the ring appears only on the main display

### Requirement: Soft glow
The system SHALL surround the ring with a soft glow in the ring's color that extends inward toward the center of the screen. The glow's strength SHALL scale with the Brightness setting.

#### Scenario: Glow around the ring
- **WHEN** the ring is on
- **THEN** a blurred glow fades out from the ring toward the center of the screen

### Requirement: On/off control
The system SHALL let the user switch the ring light off and on without quitting the app. While the ring light is off, nothing SHALL be drawn on the screen.

#### Scenario: Turn the ring off
- **WHEN** the ring is on and the user switches it off
- **THEN** the ring and its glow disappear from the screen
- **AND** the app keeps running in the menu bar

#### Scenario: Turn the ring back on
- **WHEN** the ring is off and the user switches it on
- **THEN** the ring reappears with the current thickness, color temperature and brightness

### Requirement: Ring thickness
The system SHALL draw the ring with the Thickness setting, from 10 to 100 points. Changing the thickness SHALL move the ring's inner edge while its outer edge stays in place.

#### Scenario: Thicker ring
- **WHEN** the user raises the Thickness setting
- **THEN** the ring widens toward the center of the screen
- **AND** the ring's outer edge does not move

### Requirement: Color temperature
The system SHALL color the ring according to the Temperature setting, which runs from Warm to Cool. The warm end SHALL be orange (RGB 1.0, 0.7, 0.4), the midpoint neutral white, and the cool end light blue (RGB 0.85, 0.95, 1.0), blending linearly from warm to white and from white to cool.

#### Scenario: Warm light
- **WHEN** the user moves the Temperature slider fully to Warm
- **THEN** the ring glows orange

#### Scenario: Neutral light
- **WHEN** the Temperature slider is at its midpoint
- **THEN** the ring is white

#### Scenario: Cool light
- **WHEN** the user moves the Temperature slider fully to Cool
- **THEN** the ring glows light blue

### Requirement: Brightness sets ring intensity
The system SHALL draw the ring at an opacity equal to the Brightness setting (10% to 100%), so a lower Brightness gives a dimmer, more translucent ring and glow.

#### Scenario: Dim the ring
- **WHEN** the user lowers Brightness to 30%
- **THEN** the ring is drawn at 30% opacity
- **AND** its glow dims in proportion

### Requirement: Non-intrusive overlay
The ring SHALL float above regular application windows on every desktop Space. It SHALL never take keyboard focus, and it SHALL let all mouse input pass through to the windows beneath it, including over the ring itself.

#### Scenario: Click through the ring
- **WHEN** the user clicks a spot covered by the ring or inside it
- **THEN** the click reaches the window underneath as if the ring were not there

#### Scenario: Typing is unaffected
- **WHEN** the user types while the ring is on
- **THEN** the keystrokes go to the application the user is working in

#### Scenario: Switching Spaces
- **WHEN** the user switches to another desktop Space
- **THEN** the ring is shown there as well
