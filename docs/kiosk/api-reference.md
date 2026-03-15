# Kiosk — API Reference

## UKioskSubsystem

`#include "KioskSubsystem.h"`

### Screen Stack

| Function | Signature | Description |
|----------|-----------|-------------|
| `Push` | `void Push(TSharedRef<SKioskScreen> Screen)` | Push screen onto stack |
| `Pop` | `void Pop()` | Remove top screen |
| `PopTo` | `void PopTo(TSharedRef<SKioskScreen> Screen)` | Pop until target is top |
| `Replace` | `void Replace(TSharedRef<SKioskScreen> Screen)` | Replace top screen |
| `Clear` | `void Clear()` | Remove all screens |
| `GetTopScreen` | `SKioskScreen* GetTopScreen()` | Current top screen (may be null) |
| `GetStackDepth` | `int32 GetStackDepth()` | Number of screens on stack |
| `IsEmpty` | `bool IsEmpty()` | Stack has no screens |

### Theming

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetTheme` | `void SetTheme(UKioskThemeData* Theme)` | Apply a new theme |
| `GetTheme` | `UKioskThemeData* GetTheme()` | Current active theme |

### Convenience Overlays

| Function | Signature | Description |
|----------|-----------|-------------|
| `ShowModal` | `void ShowModal(FText Title, FText Body, FSimpleDelegate OnConfirm, FSimpleDelegate OnCancel)` | Dimmed confirmation dialog |
| `ShowToast` | `void ShowToast(FText Message, float Duration)` | Auto-dismiss notification |
| `HideToast` | `void HideToast()` | Dismiss toast immediately |

## UKioskThemeData

| Property | Type | Description |
|----------|------|-------------|
| `Primary` | FLinearColor | Primary brand color |
| `Secondary` | FLinearColor | Secondary color |
| `Accent` | FLinearColor | Highlight accent |
| `Background` | FLinearColor | Screen background |
| `Text` | FLinearColor | Primary text color |
| `Muted` | FLinearColor | Muted/secondary text |
| `HeadingFont` | FSlateFontInfo | H1 heading font |
| `BodyFont` | FSlateFontInfo | Body text font |
| `CaptionFont` | FSlateFontInfo | Caption/label font |
| `ButtonHeight` | float | Standard button height |
| `Spacing` | float | Base spacing unit |
| `BorderRadius` | float | Widget corner rounding |

## Pre-Built Slate Widgets

| Widget | Header | Description |
|--------|--------|-------------|
| `SKioskButton` | KioskButton.h | Themed push button |
| `SKioskScrollBox` | KioskScrollBox.h | Themed scrollable container |
| `SKioskProgressBar` | KioskProgressBar.h | Animated fill bar |
| `SKioskModal` | KioskModal.h | Confirmation dialog |
| `SKioskToast` | KioskToast.h | Auto-dismiss notification |
| `SKioskTabBar` | KioskTabBar.h | Tab bar with content switch |
| `SKioskListView` | KioskListView.h | Virtualized item list |
| `SKioskSlider` | KioskSlider.h | Labeled range slider |
| `SKioskToggle` | KioskToggle.h | Animated on/off toggle |
| `SKioskTextInput` | KioskTextInput.h | Single-line text entry |
