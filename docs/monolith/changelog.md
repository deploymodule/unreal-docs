# Monolith — Changelog

## v0.1.0

Initial release.

### Added

**Core Infrastructure**

- `FMonolithEditorModule` — single `MonolithEditor` module with `UToolMenus`-based menu registration for both the World Outliner actor context menu and the Content Browser asset context menu
- `UMonolithSettings` — `UDeveloperSettings` subclass with 13 per-category boolean toggles and soft references to `UMonolithConfig` and `UMonolithOrganizationPreset`; accessible at **Edit > Project Settings > Plugins > Monolith Utilities**
- `UMonolithConfig` — `UDataAsset` subclass storing all numeric and string defaults for utility operations (grid spacing, randomization ranges, naming patterns, placement counts, asset zoo thresholds)
- `UMonolithOrganizationPreset` — `UDataAsset` subclass providing `ClassToFolderMap`, `ClassToLayerMap`, and `ClassToStreamingLevelMap` for class-based organization operations
- Full `FScopedTransaction` wrapping on all mutating operations, providing undo/redo support through the standard Unreal editor undo stack
- No-restart category enable/disable: toggling a category in Project Settings removes or restores its menu section immediately

**Audit Category** (`MonolithSceneAudit`)

- Find Zero Scale Actors — scans the persistent level and selects all actors with a world scale of zero on any axis; results logged to Output Log
- Find Broken Material References — selects all mesh actors with null or invalid material slots
- Find Unbound Sequencer Actors — selects actors that appear in a Level Sequence binding but no longer exist at the expected path
- Find Actors Outside World Bounds — selects actors whose location exceeds the level's configured world bounds
- Find Duplicate Actor Labels — selects actors whose display label is shared by at least one other actor in the level
- Find Actors Missing Required Tags — selects actors that do not carry any of the tags defined in `UMonolithConfig::RequiredTags`

**Collision Category** (`MonolithCollision`)

- Set Collision Preset — dialog to choose from all registered collision profiles; applies selected preset to all mesh components in the selection
- Enable Complex as Simple — sets collision complexity to `CTF_UseComplexAsSimple` on all mesh components in the selection
- Disable All Collision — sets the collision enabled flag to `NoCollision` on all mesh components in the selection

**Folders Category** (`MonolithSceneOrganize`)

- Create Folder from Selection — dialog for folder name; creates folder and moves all selected actors into it
- Move to Existing Folder — dialog listing current folders; moves selection to the chosen folder
- Organize by Class — creates subfolders named after each represented actor class and distributes the selection; uses `UMonolithOrganizationPreset::ClassToFolderMap` if an active preset is assigned
- Select All in Same Folder — expands the current selection to include every actor in the same folder as any currently selected actor

**Layers Category** (`MonolithSceneOrganize`)

- Create Layer from Selection — dialog for layer name; creates a new World layer and adds all selected actors to it
- Add to Existing Layer — dialog listing existing layers; adds selection to the chosen layer
- Organize by Class into Layers — creates or reuses layers named after each actor class and assigns actors; respects `UMonolithOrganizationPreset::ClassToLayerMap`

**Layout Category** (`MonolithLayout`)

- Arrange in Grid — dialog for row count, column count, X spacing, and Y spacing; repositions actors in a grid centered on the selection's original bounding-box midpoint
- Arrange in Circle — dialog for radius; distributes actors at equal angular intervals around a circle in the XY plane
- Arrange in Line — dialog for axis (X, Y, or Z) and spacing; positions actors end-to-end along the chosen axis
- Arrange in Arc — dialog for start angle, end angle, and radius; distributes actors along an arc in the XY plane
- Stack Vertically — dialog for per-step height; stacks actors vertically with configurable spacing
- Spread from Center — dialog for spread distance; moves each actor outward from the selection centroid by a per-actor increment

**Level Sequence Category** (`MonolithSequencerActor`)

- Add to Sequence — adds all selected actors to the currently active Level Sequence, creating Sequencer bindings and initializing transform tracks with current world transform values
- Remove from Sequence — removes selected actors' bindings from the currently active Level Sequence
- Bake to World Transform — evaluates the active sequence at the current playhead position and writes the evaluated transforms back to the actors' world transforms, then optionally removes the binding
- Set Defaults from World — updates the sequence default-value section for each selected actor's transform track to match the actor's current world transform

**Level Streaming Category** (`MonolithLevelStreaming`)

- Move to New Streaming Level — dialog for sub-level name; creates a new streaming level asset, moves selected actors into it, and registers it with the persistent level
- Move to Existing Streaming Level — dialog listing existing sub-levels; moves selection to the chosen level
- Organize by Class into Streaming Levels — creates or reuses streaming levels per actor class and distributes actors; respects `UMonolithOrganizationPreset::ClassToStreamingLevelMap`
- Merge to Persistent Level — moves all actors from a selected streaming level back into the persistent level and removes the streaming level reference
- Toggle Streaming Level Visibility — toggles editor visibility for the streaming level containing any selected actor

**LOD & Culling Category** (`MonolithLOD`)

- Force LOD Level — dialog to choose a LOD index (0–7); sets `ForcedLodModel` on all static mesh components in the selection
- Set Culling Distance — dialog for cull distance in world units; sets `LDMaxDrawDistance` on all primitive components in the selection
- Clear Forced LOD — resets `ForcedLodModel` to 0 (automatic) on all static mesh components in the selection

**Materials Category** (`MonolithMaterialActor`, `MonolithMaterialAsset`)

- Batch Apply Material — asset picker dialog; assigns the chosen material to every material slot on every mesh component in the selection
- Copy Material from First — reads all material slots from the first selected actor and applies the same assignments to all other selected actors
- Find Actors with Material Overrides — scans the selection and selects only actors whose mesh components have per-instance material overrides (different from the mesh asset's default materials); results logged
- Batch Replace Material in Assets (Content Browser) — asset picker with a "Find" material and a "Replace With" material; scans all selected assets and replaces every reference to the Find material with the Replace material

**Naming Category** (`MonolithNaming`)

- Add Prefix — dialog for prefix string; prepends to every selected actor's display label
- Add Suffix — dialog for suffix string; appends to every selected actor's display label
- Add Sequence Numbers — numbers selected actors sequentially using the format string from `UMonolithConfig::SequenceNumberFormat`; order follows Outliner order
- Find and Replace in Labels — dialog for search string and replacement string; updates all labels containing the search string
- Auto-Name from Class — renames each actor using `UMonolithConfig::AutoNamePattern` with the actor class short name; numbers reset per class
- Strip Trailing Numbers — removes trailing numeric characters (and optional preceding separator characters) from each label
- Copy Label to Tags — adds each actor's current display label as an actor tag, creating a stable string identifier that survives future renames

**Snapshot Category** (`MonolithSceneSnapshot`)

- Capture Snapshot — dialog for snapshot name; records the world transform of every selected actor into an in-memory `UMonolithSnapshotData` keyed by the snapshot name
- Restore from Snapshot — dialog listing existing snapshots; restores each selected actor's world transform to the value recorded in the chosen snapshot
- Capture Full Level Snapshot — captures the world transform of every actor in the persistent level into a single named snapshot

**Transform Category** (`MonolithTransform`)

- Randomize Position — dialog for per-axis min/max offset ranges; adds a random offset within the range to each selected actor's current location
- Randomize Rotation — dialog for per-axis min/max ranges; adds a random rotation within the range to each actor's current rotation
- Randomize Scale — dialog for uniform or per-axis min/max multiplier ranges; multiplies each actor's current scale by a random value in the range
- Snap to Ground — traces downward from each actor's origin and places the actor on the first surface hit; uses component bounds to avoid embedding actors in the floor
- Align to Surface Normal — traces from each actor and rotates the actor to align its up-vector with the surface normal at the impact point
- Distribute Evenly — dialog for axis; moves actors so that they are equally spaced along the chosen axis between the positions of the first and last actor in the selection
- Spawn Transform Gizmo Parent — creates a temporary pivot actor at the selection centroid and attaches all selected actors to it, enabling group rotation and scaling around a custom pivot
- Detach from Gizmo Parent — bakes the gizmo parent's transform into each child actor's world transform, detaches the children, and deletes the gizmo actor

**Visibility Groups Category** (`MonolithVisibilityGroup`)

- Create Group from Selection — dialog for group name; assigns an actor tag `MonolithGroup_[name]` to every selected actor
- Toggle Group Visibility — submenu listing all current visibility groups; toggles `bHiddenInGame` and editor visibility on all actors carrying the selected group tag
- Show All Groups — makes all actors carrying any `MonolithGroup_` tag visible in both the editor and game
- Select Group Members — submenu listing all current visibility groups; selects all actors in the level carrying the chosen group tag

**Dependencies Category** (`MonolithDependency`)

- Show Level Dependencies — analyzes the selected Level asset and prints a categorized dependency report (StaticMesh, Material, Texture, Blueprint, Sound, etc.) to the Output Log
- Find Unused Assets — compares assets in the right-clicked Content Browser folder against all loaded-level references; selects assets that are not referenced
- Export to CSV — saves a dependency manifest for the selected Level or folder to a user-chosen file path; columns: Asset Name, Asset Type, Path, Reference Count
