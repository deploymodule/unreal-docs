# Monolith — API Reference

## UMonolithSettings

`UMonolithSettings` is a `UDeveloperSettings` subclass. Its values are stored in `Config/DefaultGame.ini` under the `[/Script/MonolithEditor.MonolithSettings]` section. Access it at runtime via `GetDefault<UMonolithSettings>()`.

Access in the editor: **Edit > Project Settings > Plugins > Monolith Utilities**

### Category Toggles

Each toggle controls whether the corresponding category's menu section is registered with `UToolMenus` on startup. Disabling a category removes its menu entries with zero per-frame overhead. Changes take effect immediately without an editor restart.

| Property | Type | Default | Controls |
|----------|------|---------|----------|
| `bEnableAudit` | `bool` | `true` | Audit category (scene health checks) |
| `bEnableCollision` | `bool` | `true` | Collision category (batch preset operations) |
| `bEnableFolders` | `bool` | `true` | Folders category (World Outliner organization) |
| `bEnableLayers` | `bool` | `true` | Layers category (World layer management) |
| `bEnableLayout` | `bool` | `true` | Layout category (grid, circle, line, arc, stack) |
| `bEnableLevelSequence` | `bool` | `true` | Level Sequence category (Sequencer binding operations) |
| `bEnableLevelStreaming` | `bool` | `true` | Level Streaming category (sub-level organization) |
| `bEnableLOD` | `bool` | `true` | LOD & Culling category |
| `bEnableMaterials` | `bool` | `true` | Materials category (actor and asset operations) |
| `bEnableNaming` | `bool` | `true` | Naming category (label and tag management) |
| `bEnableSnapshot` | `bool` | `true` | Snapshot category (transform capture/restore) |
| `bEnableTransform` | `bool` | `true` | Transform category (randomize, snap, distribute) |
| `bEnableVisibilityGroups` | `bool` | `true` | Visibility Groups category |

### Asset References

| Property | Type | Description |
|----------|------|-------------|
| `Config` | `TSoftObjectPtr<UMonolithConfig>` | DataAsset providing all numeric and string defaults. If null, built-in defaults are used. |
| `OrganizationPreset` | `TSoftObjectPtr<UMonolithOrganizationPreset>` | DataAsset mapping actor classes to folders, layers, and streaming levels. Optional. |

---

## UMonolithConfig

`UMonolithConfig` is a `UDataAsset` subclass. Create one in the Content Browser (**Miscellaneous > Data Asset > MonolithConfig**) and assign it to `UMonolithSettings::Config`.

### Grid and Layout Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `GridSpacingX` | `float` | `200.0` | World-space spacing between columns when arranging in a grid (cm) |
| `GridSpacingY` | `float` | `200.0` | World-space spacing between rows when arranging in a grid (cm) |
| `CircleRadius` | `float` | `500.0` | Default radius for Arrange in Circle (cm) |
| `LineSpacing` | `float` | `200.0` | Default spacing for Arrange in Line (cm) |
| `ArcStartAngle` | `float` | `0.0` | Default arc start angle in degrees |
| `ArcEndAngle` | `float` | `180.0` | Default arc end angle in degrees |
| `ArcRadius` | `float` | `500.0` | Default arc radius (cm) |
| `StackHeight` | `float` | `100.0` | Vertical spacing per step for Stack Vertically (cm) |
| `SpreadDistance` | `float` | `300.0` | Outward distance per actor for Spread from Center (cm) |

### Transform Randomization Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `RandomPositionRangeX` | `FVector2D` | `(-100, 100)` | Min/max X offset for Randomize Position (cm) |
| `RandomPositionRangeY` | `FVector2D` | `(-100, 100)` | Min/max Y offset for Randomize Position (cm) |
| `RandomPositionRangeZ` | `FVector2D` | `(0, 0)` | Min/max Z offset for Randomize Position (cm) |
| `RandomRotationRangeYaw` | `FVector2D` | `(-180, 180)` | Min/max yaw for Randomize Rotation (degrees) |
| `RandomRotationRangePitch` | `FVector2D` | `(0, 0)` | Min/max pitch for Randomize Rotation (degrees) |
| `RandomRotationRangeRoll` | `FVector2D` | `(0, 0)` | Min/max roll for Randomize Rotation (degrees) |
| `RandomScaleUniform` | `FVector2D` | `(0.8, 1.2)` | Min/max uniform scale multiplier |
| `bRandomScalePerAxis` | `bool` | `false` | When true, X/Y/Z scale axes are randomized independently |
| `RandomScaleRangeX` | `FVector2D` | `(0.8, 1.2)` | Per-axis scale range for X (used when `bRandomScalePerAxis` is true) |
| `RandomScaleRangeY` | `FVector2D` | `(0.8, 1.2)` | Per-axis scale range for Y |
| `RandomScaleRangeZ` | `FVector2D` | `(0.8, 1.2)` | Per-axis scale range for Z |

### Naming Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `SequenceNumberFormat` | `FString` | `"{label}_{n:02}"` | Format string for Add Sequence Numbers; `{n}` is the counter, `:02` is the minimum width |
| `AutoNamePattern` | `FString` | `"{class}_{n:03}"` | Pattern for Auto-Name from Class; `{class}` is replaced with the actor class short name |
| `DefaultPrefix` | `FString` | `""` | Pre-filled value in the Add Prefix dialog |
| `DefaultSuffix` | `FString` | `""` | Pre-filled value in the Add Suffix dialog |

### Placement Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `PlacementGridSpacingX` | `float` | `300.0` | Column spacing for Place as Grid (cm) |
| `PlacementGridSpacingY` | `float` | `300.0` | Row spacing for Place as Grid (cm) |
| `PlacementGridColumns` | `int32` | `5` | Number of columns for Place as Grid |
| `AssetZooSmallThreshold` | `float` | `100.0` | Mesh bounding-box size threshold (cm) for "Small" category in asset zoo |
| `AssetZooLargeThreshold` | `float` | `500.0` | Mesh bounding-box size threshold (cm) for "Large" category; meshes above this are "Extra Large" |

---

## UMonolithOrganizationPreset

`UMonolithOrganizationPreset` is a `UDataAsset` subclass. Assign it to `UMonolithSettings::OrganizationPreset`. It is consulted by the "Organize by Class" operations in the Folders, Layers, and Level Streaming categories.

When an operation looks up a class, it walks the inheritance chain from the concrete class upward until it finds a match. Unmatched actors are left in place.

| Property | Key Type | Value Type | Description |
|----------|----------|------------|-------------|
| `ClassToFolderMap` | `TSubclassOf<AActor>` | `FString` | Maps an actor class to a World Outliner folder path (e.g., `"Props/Furniture"`) |
| `ClassToLayerMap` | `TSubclassOf<AActor>` | `FName` | Maps an actor class to a World layer name |
| `ClassToStreamingLevelMap` | `TSubclassOf<AActor>` | `FSoftObjectPath` | Maps an actor class to an existing streaming level asset path |

---

## UMonolithSnapshotData

`UMonolithSnapshotData` is a `UObject` subclass (not a DataAsset) used internally by the Snapshot category. It is stored in memory during the editor session and is not automatically persisted to disk. The Snapshot category's "Capture Full Level" operation populates a `UMonolithSnapshotData` with transforms for every actor in the persistent level.

| Field | Type | Description |
|-------|------|-------------|
| `SnapshotName` | `FName` | Unique identifier for this snapshot, entered by the user in the capture dialog |
| `ActorTransforms` | `TMap<TSoftObjectPtr<AActor>, FTransform>` | Maps each captured actor to its world transform at capture time |
| `CaptureTimestamp` | `FDateTime` | Editor time at which the snapshot was taken |

!!! note
    Snapshots are stored in a session-scoped registry (`FMonolithSnapshotRegistry`) that lives for the duration of the editor session. Closing and reopening the editor clears all snapshots. For persistent transform states, use Sequencer default values or a saved Level.

---

## Utility Namespaces

All Monolith utility logic is implemented in C++ namespaces within the `MonolithEditor` module. These are not public APIs intended for direct external calls; they are documented here to aid in understanding which source files govern each category's behavior and to support contributors or technical artists extending the plugin.

| Namespace | Header | Governed Categories / Operations |
|-----------|--------|----------------------------------|
| `MonolithSceneAudit` | `MonolithSceneAudit.h` | Audit — zero-scale, broken materials, unbound sequences, world bounds, duplicate labels, missing tags |
| `MonolithCollision` | `MonolithCollision.h` | Collision — preset assignment, complex-as-simple, disable all |
| `MonolithSceneOrganize` | `MonolithSceneOrganize.h` | Folders and Layers — create folder, organize by class, layer assignment |
| `MonolithLayout` | `MonolithLayout.h` | Layout — grid, circle, line, arc, stack, spread |
| `MonolithLevelStreaming` | `MonolithLevelStreaming.h` | Level Streaming — move to level, organize by class, merge, visibility toggle |
| `MonolithSequencerActor` | `MonolithSequencerActor.h` | Level Sequence — add to sequence, remove, bake transforms, set defaults |
| `MonolithLOD` | `MonolithLOD.h` | LOD & Culling — force LOD, set cull distance, clear forced LOD |
| `MonolithMaterialActor` | `MonolithMaterialActor.h` | Materials (actor-side) — batch apply, copy from first, find overrides |
| `MonolithMaterialAsset` | `MonolithMaterialAsset.h` | Materials (asset-side) — batch replace material references in Content Browser |
| `MonolithNaming` | `MonolithNaming.h` | Naming — prefix, suffix, sequence numbers, find/replace, auto-name, strip numbers, copy label to tags |
| `MonolithSceneSnapshot` | `MonolithSceneSnapshot.h` | Snapshot — capture, restore, full-level snapshot |
| `MonolithTransform` | `MonolithTransform.h` | Transform — randomize, snap to ground, align to surface, distribute evenly, gizmo parent |
| `MonolithVisibilityGroup` | `MonolithVisibilityGroup.h` | Visibility Groups — create, toggle, show all, select members |
| `MonolithActorPlacement` | `MonolithActorPlacement.h` | Placement — place as grid, asset zoo |
| `MonolithDependency` | `MonolithDependency.h` | Dependencies — show by type, find unused, export to CSV |

### Menu Registration

All menus are registered by `FMonolithEditorModule::StartupModule` through `UToolMenus`. The extension points used are:

```cpp
// World Outliner actor context menu
UToolMenus::Get()->ExtendMenu(TEXT("LevelEditor.ActorContextMenu"));

// Content Browser asset context menu
UToolMenus::Get()->ExtendMenu(TEXT("ContentBrowser.AssetContextMenu"));
```

Each category that is enabled in `UMonolithSettings` adds a named section to the Monolith submenu. Sections are skipped (not registered) for disabled categories, keeping the menu structure clean.

### Transaction Pattern

Every mutating operation follows this pattern:

```cpp
FScopedTransaction Transaction(LOCTEXT("MonolithOperationName", "Monolith: Operation Description"));

for (AActor* Actor : SelectedActors)
{
    Actor->Modify();
    // ... apply change ...
}
```

The `FScopedTransaction` is opened before the loop, and `Actor->Modify()` is called on each actor before any property is changed. This ensures the undo system captures the pre-change state correctly and that Ctrl+Z reverses the entire batch as a single step.
