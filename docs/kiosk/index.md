# Kiosk

Comprehensive data-driven UI framework — pure Slate, Data Asset themes, and `UGameInstanceSubsystem` screen stack.

| | |
|---|---|
| **Category** | UI |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **UMG Required** | No (pure Slate) |

Kiosk is a complete UI framework built on pure Slate (no UMG dependency). It provides a managed screen stack, a Data Asset-driven theming system, and a library of pre-built Slate widgets styled by theme. Screens are pushed and popped on the stack — only the top screen receives input, and screens can be transparent/overlay. Themes define colors, fonts, spacing, and widget styles centrally, so the entire UI appearance can be changed by swapping a single Data Asset. Ideal for games that need a polished, consistent UI without the overhead of UMG Blueprint compilation.

## In This Section

- [Overview](overview.md) — Screen stack, theming, and widget architecture
- [Usage](usage.md) — Creating screens, pushing them, and applying themes
- [API Reference](api-reference.md) — Classes, functions, and Slate widget catalog
- [Changelog](changelog.md) — Version history
