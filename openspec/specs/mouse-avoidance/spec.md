# mouse-avoidance Specification

## Purpose
Keeps the ring light from hiding what the user is pointing at by making the ring transparent around the mouse pointer.

## Requirements

### Requirement: Transparent area around the pointer
While Avoid Mouse is enabled, the system SHALL hide the ring and its glow inside a circle with a 60-point radius centered on the mouse pointer. The circle SHALL follow the pointer as it moves, including while another application is active.

#### Scenario: Pointer over the ring
- **WHEN** Avoid Mouse is enabled and the user moves the pointer onto the ring
- **THEN** the part of the ring within 60 points of the pointer becomes transparent
- **AND** the rest of the ring stays lit

#### Scenario: Pointer moves away
- **WHEN** the pointer moves off the ring
- **THEN** the part of the ring that was hidden is lit again

#### Scenario: Another application is active
- **WHEN** another application is focused and the user moves the pointer onto the ring
- **THEN** the ring still becomes transparent around the pointer

### Requirement: Avoid Mouse toggle
The system SHALL provide an Avoid Mouse checkbox in the control panel that turns the transparent area on and off. Avoid Mouse SHALL be enabled when the app launches.

#### Scenario: Disable Avoid Mouse
- **WHEN** the user unchecks Avoid Mouse
- **THEN** the full ring is drawn wherever the pointer is

#### Scenario: Enabled on launch
- **WHEN** the app launches
- **THEN** Avoid Mouse is checked
