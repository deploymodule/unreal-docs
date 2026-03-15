# Kiosk — Changelog

## v1.0.0 — Initial Release

### Added
- `UKioskSubsystem` — `UGameInstanceSubsystem` managing a Slate screen stack
- Screen stack operations: Push, Pop, PopTo, Replace, Clear
- Top-screen input isolation — only the top screen receives input
- `UKioskThemeData` — Data Asset defining the complete UI visual language
- Runtime theme switching — `SetTheme` instantly re-renders all screens
- `ShowModal` — dimmed-background confirmation dialog with confirm/cancel delegates
- `ShowToast` — auto-dismissing notification overlay
- `SKioskButton` — themed button with hover, press, and disabled states
- `SKioskScrollBox` — themed scrollable container
- `SKioskProgressBar` — animated fill bar with configurable color zones
- `SKioskModal` — confirmation dialog Slate widget
- `SKioskToast` — timed notification Slate widget with fade animation
- `SKioskTabBar` — horizontal tab bar with animated active indicator
- `SKioskListView` — virtualized list with typed item templates
- `SKioskSlider` — labeled range slider with formatted value display
- `SKioskToggle` — animated on/off toggle switch
- `SKioskTextInput` — single-line text input with validation callback
- `UKioskScreenBase` — Blueprint-accessible screen wrapper
- Full Blueprint-callable subsystem API
- Pure Slate implementation — no UMG dependency
- Input processor registration for proper UI/gameplay input separation
