# Monolith

Comprehensive scene management and production workflow plugin providing 80+ context-menu utilities for organizing, auditing, transforming, and batch-operating on actors and assets directly in the Unreal Editor.

| | |
|---|---|
| **Category** | Virtual Production |
| **UE Version** | 5.7+ |
| **Target** | Editor only |
| **Runtime Impact** | None |
| **Module** | `MonolithEditor` |

Monolith is an editor-only plugin that adds a "Monolith" submenu to the World Outliner right-click context menu and to the Content Browser right-click context menu. All operations are organized into 15 functional categories and wrapped in `FScopedTransaction`, giving you full undo/redo support for every action. Individual categories can be disabled without restarting the editor, eliminating menu clutter for operations you do not use.

## Features

- Scene auditing: find zero-scale actors, broken material references, stale Sequencer bindings, out-of-bounds actors, duplicate labels, and missing tags
- Batch naming: add prefixes and suffixes, sequential numbering, find-and-replace, auto-name from class, and strip trailing numbers
- Spatial layout: arrange actors in grids, circles, lines, arcs, vertical stacks, or spread outward from a center point
- Transform tools: randomize position, rotation, and scale; snap to ground; align to surface normal; distribute evenly along an axis
- Folder and layer organization: create folders from selections, organize actors by class, and manage World layers
- Level Streaming workflow: move actors to new or existing streaming levels, organize by class into separate sub-levels, and merge sub-levels back
- Lightweight snapshots: capture and restore named transform states for individual actors or entire levels
- Content Browser utilities: batch material replacement, dependency audit, unused-asset detection, and grid placement of assets in the scene
- Fully configurable via `UMonolithSettings` (category toggles), `UMonolithConfig` DataAsset (numeric defaults), and `UMonolithOrganizationPreset` DataAsset (class-to-folder/layer/streaming-level mappings)

UE 5.7 · Editor only · Virtual Production

## In This Section

- [Overview](overview.md) — Architecture, categories, and configuration system
- [Usage](usage.md) — Installation, accessing utilities, and worked examples for common tasks
- [API Reference](api-reference.md) — Settings classes, config properties, and utility namespaces
- [Changelog](changelog.md) — Version history
