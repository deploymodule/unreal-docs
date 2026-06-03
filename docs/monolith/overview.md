# Monolith — Overview

## What It Is

Monolith is the scene-management and batch-operations layer for Unreal Engine production workflows. Rather than requiring custom Blueprint scripts or manual repetition for common production tasks — renaming a hundred props, snapping a set of actors to a surface, moving selected actors into a streaming level — Monolith exposes all of these operations as right-click context menu entries that are always one or two clicks away in the editor.

The plugin is editor-only and has no runtime module. Every operation targets the currently selected actors in the World Outliner or the currently selected assets in the Content Browser, operates immediately, and registers an undo transaction so that Ctrl+Z works as expected.

Monolith is designed for medium-to-large production scenes where scene hygiene, iteration speed, and organizational consistency matter: virtual production stages, large environment sets, cinematic sequences with dozens of bound actors, and content libraries with hundreds of assets.

## Module Layout

Monolith is a single-module plugin. There is no runtime module and no dependency on any Monolith code at package time.

| Module | Type | Description |
|--------|------|-------------|
| `MonolithEditor` | Editor | All menu registration, utility implementations, settings classes, and DataAsset types |

The module entry point is `FMonolithEditorModule`, which calls `UToolMenus` at startup to register all Monolith submenus into the World Outliner context menu (`LevelEditor.ActorContextMenu`) and the Content Browser context menu (`ContentBrowser.AssetContextMenu`). Category entries are registered conditionally based on the toggle values in `UMonolithSettings`, so disabling a category in Project Settings removes its menu section entirely without any overhead.

## Access Points

### World Outliner Context Menu

Right-click any actor or multi-selection of actors in the World Outliner. The **Monolith** submenu appears at the bottom of the context menu. Hover over it to expand the category list, then hover over a category to see its operations.

All operations in the World Outliner context menu act on the current selection. If an operation requires a minimum number of actors (for example, distributing evenly requires at least two), it is either greyed out or produces a notification explaining the requirement.

### Content Browser Context Menu

Right-click any asset or multi-selection of assets in the Content Browser. The **Monolith** submenu appears and shows only the categories that operate on assets: **Materials**, **Dependencies**, and **Placement**.

## Utility Categories

| Category | Description | Key Operations |
|----------|-------------|----------------|
| **Audit** | Scene health checks that surface common production mistakes | Find zero-scale actors, find broken material references, find unbound Sequencer actors, find actors outside world bounds, find duplicate labels, find actors missing required tags |
| **Collision** | Batch-set collision presets and complexity across a selection | Set collision preset (BlockAll, OverlapAll, NoCollision, etc.), enable complex-as-simple, disable all collision |
| **Folders** | Create and manage World Outliner folder organization | Create folder from selection, move to existing folder, organize by class into subfolders, select all actors in same folder |
| **Layers** | Manage World layer membership | Create layer from selection, add selection to existing layer, organize actors by class into layers |
| **Layout** | Spatially arrange actors using procedural patterns | Arrange in Grid, Arrange in Circle, Arrange in Line, Arrange in Arc, Stack Vertically, Spread from Center |
| **Level Sequence** | Batch Sequencer binding and track operations | Add actors to sequence with transform tracks, remove actors from sequence, bake evaluated transforms to world, set default values from world transforms |
| **Level Streaming** | Move and organize actors across streaming levels | Move to new streaming level, move to existing streaming level, organize by class into streaming levels, merge sub-level to persistent, toggle streaming level visibility |
| **LOD & Culling** | Manage level-of-detail settings on mesh actors | Force specific LOD level, set culling distance, clear forced LOD |
| **Materials** | Batch material assignment and discovery | Batch apply material to all selected meshes, copy material from first to all, find actors with material overrides, batch replace material references in assets (Content Browser) |
| **Naming** | Actor label and tag management | Add prefix, add suffix, add sequence numbers, find and replace in labels, auto-name from class, strip trailing numbers, copy label to tags |
| **Snapshot** | Lightweight transform capture and restore | Capture named snapshot of selected actors, restore selection from snapshot, full-level snapshot |
| **Transform** | Spatial manipulation beyond the standard transform widget | Randomize position/rotation/scale, snap to ground, align to surface normal, distribute evenly along axis, spawn transform gizmo parent, detach from gizmo parent |
| **Visibility Groups** | Tag-based visibility grouping for fast scene control | Create visibility group from selection, toggle group visibility, show all groups, select all group members |
| **Dependencies** | Content assessment from the Content Browser | Show level dependencies by asset type, find unused assets in folder, export dependency list to CSV |
| **Placement** | Place Content Browser assets into the scene in bulk | Place as grid, create asset zoo organized by mesh size |

## Configuration System

Monolith exposes three configuration objects. None require an editor restart when changed.

### UMonolithSettings

`UMonolithSettings` is the plugin's `UDeveloperSettings` subclass, accessible at **Edit > Project Settings > Plugins > Monolith Utilities**. It contains one boolean toggle per category group. When a toggle is disabled, the corresponding menu section is not registered with `UToolMenus` at all, meaning there is zero menu overhead for categories you do not use.

It also holds a soft object reference to the active `UMonolithConfig` DataAsset, and an optional reference to a `UMonolithOrganizationPreset` DataAsset for class-based organization operations.

### UMonolithConfig

`UMonolithConfig` is a `UDataAsset` subclass you create in your Content Browser and assign in `UMonolithSettings`. It stores all numeric and string defaults used by the utility operations: grid spacing values, randomization ranges, sequence number format strings, naming patterns, layout radii, and so on.

Creating a project-specific config lets you lock in values appropriate for your scene scale (for example, centimeter vs. meter-scale randomization ranges) without modifying plugin source.

### UMonolithOrganizationPreset

`UMonolithOrganizationPreset` is a `UDataAsset` subclass that maps actor classes to default destinations for the "Organize by Class" family of operations. It contains three maps:

| Map | Key | Value |
|-----|-----|-------|
| `ClassToFolderMap` | `UClass*` | Folder path string |
| `ClassToLayerMap` | `UClass*` | Layer name string |
| `ClassToStreamingLevelMap` | `UClass*` | Streaming level asset path |

When an "Organize by Class" operation runs, it looks up each actor's class (walking the inheritance chain) in the active preset's maps and moves the actor to the matching destination. Actors whose class is not in the map are left in place.

## Undo/Redo Support

Every operation in Monolith opens a named `FScopedTransaction` before modifying any actor or asset state. This means all Monolith operations integrate with the standard editor undo stack and are reversible with **Ctrl+Z**. Multi-step operations (for example, moving 200 actors into a new folder) are grouped as a single undo transaction so one Ctrl+Z reverses the entire batch.

!!! note
    Operations that only read state — Audit checks, dependency listing, and CSV export — do not open a transaction because they do not modify anything.
