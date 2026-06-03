# Object Mixer Creator

Visual editor for creating and managing Object Mixer filter Blueprints from actor class reflection — no code required.

| | |
|---|---|
| **Category** | Editor Tools |
| **UE Version** | 5.7+ |
| **Target** | Editor only |
| **Runtime Impact** | None |
| **Requires** | Object Mixer (built-in UE5) |

Object Mixer Creator is a companion tool for Unreal Engine's built-in Object Mixer panel. It removes the need to author filter Blueprint classes by hand: select actors in the viewport, check the properties you want exposed as columns, name the filter, and click Create. The plugin generates a Blueprint filter class and a config DataAsset at your configured output path and syncs the Content Browser to both assets.

## Features

- Select actors in the viewport to auto-populate their classes into the panel with a single click
- Expand each class to see all editable properties grouped by category, with a live search box for quick navigation
- Check individual properties to expose them as Object Mixer columns
- Create New mode generates a fresh filter Blueprint and config DataAsset in one step
- Update Existing mode rewrites only the config DataAsset — the Blueprint does not need to recompile
- Delete action removes both the filter Blueprint and its config DataAsset together
- Output path, default filter name, base class restriction, and property inclusion rules are all configurable in Project Settings

## In This Section

- [Overview](overview.md) — Architecture, generated assets, modes, and property discovery
- [Usage](usage.md) — Installation, step-by-step workflows, and output configuration
- [API Reference](api-reference.md) — C++ classes, functions, and settings fields
- [Changelog](changelog.md) — Version history
