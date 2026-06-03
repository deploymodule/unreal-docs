# Gaffer

Unified C++/Slate editor panel for finding, organizing, inspecting, and keyframing every camera, light, and environment actor in your scene.

| | |
|---|---|
| **Category** | Virtual Production |
| **UE Version** | 5.7+ |
| **Target** | Editor only |
| **Runtime Impact** | None |
| **Replaces** | BP CameraController, BP LightController |

Gaffer is a feature-complete scene-actor management tool built entirely in C++ and Slate. It replaces the original Blueprint-based CameraController and LightController editor utility widgets with a single, dockable panel that covers cameras and lights together, adds structured keyframing integrated with Sequencer, and supports any actor class you register with it. Because it is editor-only, it has no footprint in packaged builds.

## Features

- Filterable actor tree covering cameras, lights, environment actors, and custom registered classes
- Camera Inspector with eight keyframeable properties and a collapsible Advanced section
- Light Inspector with conditional shape sections that adapt to spot, point, rect, and directional light types
- Per-property keyframe buttons with four key modes, three interpolation modes, and three track-targeting strategies
- Auto-key mode that keys every value change automatically against the active Sequencer playhead
- Named actor groups with color, intensity scale, temperature offset, and color tint overrides
- Tag system with floating input dialog and tree filtering by tag
- Working Sets for saving and recalling actor selections
- Smart Filters by class, tag, group, or custom criteria
- Property Matrix for multi-actor property editing
- Coverage Map showing which actors are keyframed in the current sequence
- Environment Presets for saving and restoring environment lighting configurations
- Light Rigs for grouped light setups with stored transforms
- Custom class registration API for extending the actor tree to any `AActor` subclass

## In This Section

- [Overview](overview.md) — Architecture, module layout, inspector system, and keyframing pipeline
- [Usage](usage.md) — Installation, panel navigation, inspecting cameras and lights, groups, tags, and working sets
- [API Reference](api-reference.md) — Enums, classes, and key function signatures
- [Changelog](changelog.md) — Version history
