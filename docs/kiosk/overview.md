# Kiosk — Overview

## Architecture

```
UKioskSubsystem  (UGameInstanceSubsystem)
    ├── TArray<TSharedPtr<SKioskScreen>>  ScreenStack
    ├── UKioskThemeData*                  ActiveTheme
    └── TSharedPtr<SOverlay>             RootOverlay  — added to viewport

SKioskScreen  (SCompoundWidget)
    ├── virtual TSharedRef<SWidget> BuildContent()
    ├── virtual void OnPush()
    ├── virtual void OnPop()
    └── bool bBlocksInput
```

## Screen Stack

The screen stack works like a browser history:

- `Push(Screen)` — adds a screen on top, previous screen is still visible below (if screens are translucent)
- `Pop()` — removes the top screen, previous screen resumes input
- `PopTo(Screen)` — pop until a specific screen is on top
- `Replace(Screen)` — replace the top screen (no history entry)
- `Clear()` — remove all screens

Only the top screen receives input. Lower screens can still render (useful for modal overlays, confirmation dialogs, etc.).

## Theming System

A `UKioskThemeData` Data Asset defines the complete visual language of the UI:

```
UKioskThemeData
    ├── FLinearColor Primary, Secondary, Accent, Background, Text, Muted
    ├── FSlateFontInfo HeadingFont, BodyFont, CaptionFont
    ├── float BorderRadius, Spacing, ButtonHeight
    ├── FSlateColor ButtonNormal, ButtonHover, ButtonPressed
    └── UTexture2D* Icons (close, back, check, etc.)
```

All Kiosk widgets read from the active theme automatically. Switching themes at runtime (e.g., color blind mode) requires a single `UKioskSubsystem::SetTheme(NewTheme)` call.

## Pre-Built Widgets

Kiosk ships a library of common game UI widgets built entirely in Slate:

| Widget | Description |
|--------|-------------|
| `SKioskButton` | Themed button with hover and press states |
| `SKioskScrollBox` | Themed scrollable container |
| `SKioskProgressBar` | Animated fill bar with color zones |
| `SKioskModal` | Dimmed-background confirmation dialog |
| `SKioskToast` | Transient notification (auto-dismiss timer) |
| `SKioskTabBar` | Horizontal tab bar with content switching |
| `SKioskListView` | Virtualized list with item templates |
| `SKioskSlider` | Labeled range slider |
| `SKioskToggle` | Animated on/off toggle |
| `SKioskTextInput` | Single-line text entry with validation |

## Input Management

When a screen is on top, Kiosk registers an input processor that routes keyboard/gamepad input to the screen before UE's default input. This ensures UI inputs don't bleed into gameplay. When the stack is empty, input returns to normal gameplay routing.
