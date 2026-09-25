# menu-bar-controls Specification

## Purpose
Gives RingLight a menu bar presence and a compact control panel for switching the ring light on and off, adjusting it, and quitting, without a Dock icon or main window.

## Requirements

### Requirement: Menu bar app
The system SHALL run as a menu bar utility: it SHALL show an icon in the menu bar and SHALL NOT show a Dock icon or a main window.

#### Scenario: App launched
- **WHEN** the app launches
- **THEN** a rounded-rectangle icon appears in the menu bar
- **AND** no Dock icon or app window is shown

#### Scenario: Menu bar appearance changes
- **WHEN** the menu bar switches between light and dark appearance
- **THEN** the icon changes color to stay legible

### Requirement: Control panel
Clicking the menu bar icon SHALL open a control panel below it. Clicking the icon again, or clicking anywhere outside the panel, SHALL close it. From top to bottom the panel SHALL show: the camera preview; a "Ring Light" on/off switch; Brightness, Temperature and Thickness sliders; an Avoid Mouse checkbox; and a footer with a "Star on GitHub" link and a Quit button.

#### Scenario: Open the panel
- **WHEN** the panel is closed and the user clicks the menu bar icon
- **THEN** the control panel opens below the icon

#### Scenario: Close the panel
- **WHEN** the panel is open and the user clicks the menu bar icon or anywhere outside the panel
- **THEN** the panel closes

### Requirement: Adjustment controls
The control panel SHALL apply each setting to the ring immediately as it changes:
- Ring Light switch: turns the ring light on or off
- Brightness slider: 10% to 100%, labeled with the current percentage
- Temperature slider: a gradient from Warm to Cool
- Thickness slider: 10 to 100 px, labeled with the current value
- Avoid Mouse checkbox: turns mouse avoidance on or off

#### Scenario: Adjust thickness
- **WHEN** the user drags the Thickness slider
- **THEN** its label shows the new value in px
- **AND** the ring's thickness changes while the user drags

#### Scenario: Switch the ring off from the panel
- **WHEN** the user turns off the Ring Light switch
- **THEN** the ring disappears from the screen

### Requirement: Launch defaults
Every launch SHALL start from the default settings, regardless of changes made in earlier sessions: ring light on, Thickness 45 px, Temperature at its neutral midpoint, and Avoid Mouse enabled. Brightness SHALL start at the display's current brightness (see display-brightness).

#### Scenario: First launch
- **WHEN** the app launches
- **THEN** the ring is on, 45 px thick, neutral white, with Avoid Mouse enabled

#### Scenario: Relaunch after changes
- **WHEN** the user changes settings, quits, and launches the app again
- **THEN** the settings are back to their defaults

### Requirement: Keyboard shortcuts
While RingLight has keyboard focus (for example, after the user clicks inside its open control panel), pressing Space SHALL switch the ring light on or off and pressing Escape SHALL quit the app. The shortcuts SHALL NOT capture key presses while another application has keyboard focus.

#### Scenario: Toggle with Space
- **WHEN** the control panel is open and has keyboard focus and the user presses Space
- **THEN** the ring light switches on or off

#### Scenario: Quit with Escape
- **WHEN** the control panel is open and has keyboard focus and the user presses Escape
- **THEN** the app quits

#### Scenario: Another application has focus
- **WHEN** another application has keyboard focus and the user presses Space or Escape
- **THEN** the key press goes to that application
- **AND** the ring light is unaffected

### Requirement: Quit and project link
The control panel SHALL include a Quit button that quits the app, and a "Star on GitHub" link that opens the project page (https://github.com/itsOmSarraf/ringlight) in the default web browser.

#### Scenario: Quit from the panel
- **WHEN** the user clicks Quit
- **THEN** the app quits and the ring disappears

#### Scenario: Open the GitHub page
- **WHEN** the user clicks "Star on GitHub"
- **THEN** the project page opens in the default web browser
