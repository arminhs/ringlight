# Spec Delta

## Purpose

Defines which Macs and macOS versions RingLight runs on, so that it reaches Macs that cannot run macOS Tahoe's built-in Edge Light.

## ADDED Requirements

### Requirement: Minimum macOS version
The system SHALL run on macOS 12 Monterey and every later macOS version. It SHALL declare macOS 12.0 as its minimum system version, so that earlier versions of macOS refuse to open it with a clear message instead of starting it.

#### Scenario: Launch on macOS 12
- **WHEN** the user opens RingLight on a Mac running macOS 12 Monterey
- **THEN** the app starts as a menu bar app and draws the ring light

#### Scenario: Launch on macOS 11 or earlier
- **WHEN** the user tries to open RingLight on macOS 11 Big Sur or earlier
- **THEN** macOS refuses to open it and reports that it requires macOS 12.0 or later

### Requirement: Same features on every supported version
On every supported macOS version, every feature SHALL behave as its specification describes (ring light, mouse avoidance, menu bar controls, display brightness and camera preview), and every control in the control panel SHALL show its icon and label.

#### Scenario: Control panel on macOS 12
- **WHEN** the user opens the control panel on macOS 12
- **THEN** the Brightness, Temperature and Thickness controls each show their icon and label
- **AND** every control works as it does on newer macOS versions

#### Scenario: Mouse avoidance on macOS 12
- **WHEN** Avoid Mouse is enabled on macOS 12 and the user moves the pointer onto the ring
- **THEN** the ring becomes transparent around the pointer

### Requirement: Apple silicon and Intel Macs
The system SHALL run natively on both Apple silicon and Intel Macs.

#### Scenario: Intel Mac
- **WHEN** the user opens a downloaded RingLight build on an Intel Mac running macOS 12 or later
- **THEN** the app opens and runs

#### Scenario: Apple silicon Mac
- **WHEN** the user opens a downloaded RingLight build on an Apple silicon Mac
- **THEN** the app runs natively rather than under Rosetta
