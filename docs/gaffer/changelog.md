# Gaffer — Changelog

## v0.1.0 — Initial Release

### Added

**Core Architecture**

- Two-module structure: `Gaffer` (editor data model) and `GafferEditor` (Slate UI and sequencer bridge)
- `GafferTypes.h` with enums `EGafferKeyMode`, `EGafferInterpMode`, and `EGafferTrackTarget`
- `GafferKeyframeState` shared state object carrying key mode, interpolation mode, track target, and auto-key flag
- `FGafferActorRegistry` singleton for registering actor classes and property sets
- `GafferPropertySet` base class with `AddProperty`, `RemoveProperty`, and `GetProperties`
- `GafferTagRegistry` keyed by actor GUID with add, remove, query, and enumerate operations
- `GafferBulkActions` static class with batch key, reset, group assign, tag, and Sequencer select operations

**Actor Tree**

- Filterable and searchable actor tree listing cameras, lights, and environment actors by category
- Group filter dropdown to show only actors belonging to a named group
- Tag filter dropdown to show only actors carrying a specific tag
- Context menu on actor rows: Groups submenu (Add to / Remove from), Tags submenu (Add Tag / Remove Tag), Working Set actions
- Multi-selection with Property Matrix activation

**Camera Inspector**

- Eight keyframeable properties: Focal Length, Aperture, ISO, Shutter Speed, Exposure Compensation, Focus Distance, Sensor Width, Sensor Height
- Collapsible Advanced section with Blade Count
- Status row displaying 24×24 class icon, actor label, and green Bound pill when actor is in an active sequence

**Light Inspector**

- Base properties for all light types: Intensity, Attenuation Radius, Color (interactive HSV picker), Use Temperature toggle, Temperature
- Conditional Shape section adapting to light type: Spot (Inner / Outer Cone Angle), Point (Source Radius), Rect (Source Width / Source Height), Directional (Light Source Angle)
- Collapsible Advanced section with Indirect Lighting Intensity, Volumetric Scattering Intensity, and Casts Shadows toggle

**Keyframing**

- `SGafferKeyframePropertyRow` per-row widget with left circle indicator and right diamond key button
- `GafferSequencerBridge::KeyProperty` integrating with the Sequencer / MovieScene API
- `CurrentFrame` mode: keys at the Sequencer playhead
- `FirstFrameOnly` mode: keys at frame 0 regardless of playhead
- `MatchingValue` mode: updates existing keys whose value matches the current property value
- `AllSelected` mode: keys all tree-selected actors in a single operation
- `Cubic` interpolation via auto-spline tangents
- `Linear` interpolation via straight-line tangents
- `Constant` (step/hold) interpolation
- `First`, `Last`, and `Selected` track targeting strategies
- Auto-key toggle: keys every `OnValueChanged` event automatically without a button press
- Keyframe settings bar above the inspector with dropdowns for key mode, interpolation, track target, and auto-key toggle
- `SGafferInspector::BuildStatusRow` and `BuildKeyframeSettingsBar` shared across camera and light inspectors
- `SGafferInspector::AutoKeyProperty` checking `bAutoKeyEnabled` before delegating to the bridge

**Groups**

- Named actor groups stored in `GafferGroupRegistry`
- Group color swatch for visual identification in the tree
- Per-group Intensity Scale multiplier applied to all lights in the group
- Per-group Temperature Offset (Kelvin) applied to all lights in the group
- Per-group Color Tint multiplier applied to all lights in the group
- Inline group editor in the inspector panel when a group row is selected
- Context menu Add to Group / Remove from Group submenus

**Tags**

- Floating tag input dialog (`SWindow` with a single text input field) replacing hardcoded tag names
- Per-actor tag assignment stored in `GafferTagRegistry` keyed by actor GUID
- Tree filtering by tag via the Tag filter dropdown

**Working Sets**

- Named saved selections of actors
- Save current tree selection as a named Working Set
- Recall a Working Set to restore the selection
- Delete Working Sets from the Working Sets bar

**Smart Filters**

- Filter actor tree by class, tag, group, or custom criteria
- Filters compose: class + tag + group filters apply simultaneously

**Property Matrix**

- Multi-actor property editing activated when two or more actors are selected in the tree
- Shared editing across heterogeneous actor types (common properties shown, type-specific properties hidden)

**Coverage Map**

- Visual overview of which actors in the tree have at least one keyframe in the currently open sequence
- Per-actor coverage row showing keyframed properties as colored spans across the sequence timeline

**Environment Presets**

- Save current environment lighting configuration (sky light, atmospheric, fog settings) as a named preset
- Load a saved preset to restore all environment settings at once
- Delete and rename presets

**Light Rigs**

- Named grouped light setups with stored actor transforms
- Apply a rig to snap all grouped lights back to their stored transform positions
- Store current transforms into an existing rig

**Custom Class Registration**

- `FGafferActorRegistry::RegisterClass` to add any `AActor` subclass to the tree with a category label and icon
- `FGafferActorRegistry::RegisterPropertySet` to associate a `UGafferPropertySet` subclass with a registered actor class
- `UGafferPropertySet` base class with `GetProperties` virtual override for declaring custom keyframeable properties

**Code Quality**

- All event bindings use `AddSP` (shared pointer safe delegates) — no raw `this` captures
- `FScopedTransaction` placed outside loops for correct undo batching
- Context menus built via static `BuildContextMenu` function with full actor selection to avoid unsafe casts
- `GafferPropertySet::AddProperty` calls `Modify()` to correctly dirty the owning object
