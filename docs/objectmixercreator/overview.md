# Object Mixer Creator — Overview

## What It Is

Object Mixer Creator is an editor-only plugin that generates Object Mixer filter Blueprints from actor class reflection. It does not replace or extend Object Mixer itself — it is a dedicated authoring tool that produces the filter assets Object Mixer consumes.

Unreal Engine's Object Mixer is a built-in multi-actor property inspection and editing panel. To focus it on a specific set of actor classes and properties, you provide it with a filter Blueprint that subclasses `UObjectMixerObjectFilter` and overrides two functions: `GetObjectClassesToFilter` (which classes to show) and `GetColumnsToShowByDefault` (which property columns to include). Authoring these Blueprints by hand requires knowing the exact property names, navigating the Blueprint editor, and recompiling after every change to the column list. Object Mixer Creator automates all of that.

The plugin opens as a nomad dockable tab from Main Menu > Window > "Object Mixer Filter Creator". After generating a filter, you open Object Mixer separately (Main Menu > Window > Object Mixer) and select your new filter from its filter dropdown. The two tools do not share state — Object Mixer Creator is only active when you need to create or revise a filter.

## Module Layout

Object Mixer Creator is a single-module plugin.

| Module | Type | Description |
|--------|------|-------------|
| `ObjectMixerCreator` | Editor | All plugin logic: Slate UI, asset factory, base filter class, config DataAsset, and Project Settings. |

Because the module type is `Editor`, it compiles only for editor targets and has no footprint in packaged builds.

## Generated Assets

Each filter consists of two assets created together at the configured output path.

### Filter Blueprint

The Blueprint is named `U[FilterName]Filter` and subclasses `UObjectMixerCreatorFilter`. It overrides two functions:

- `GetObjectClassesToFilter()` — reads `ClassesToShow` from the config DataAsset and returns those classes to Object Mixer
- `GetColumnsToShowByDefault()` — reads `Columns` from the config DataAsset and returns those property names to Object Mixer

The Blueprint itself contains no hardcoded class or property data. All data lives in the config DataAsset, which is why updating the column list never requires a recompile.

### Config DataAsset

The DataAsset is named `[FilterName]Config` and is an instance of `UObjectMixerFilterConfig`. It stores two arrays:

- `ClassesToShow` — `TArray<TSoftClassPtr<AActor>>` listing the actor classes the filter targets
- `Columns` — `TArray<FName>` listing the property names exposed as columns

Because the config DataAsset is a plain data asset, it can be edited directly in the Details panel as well as through the Object Mixer Creator panel.

### Inheritance Relationship

```
UObjectMixerObjectFilter          (Unreal built-in)
    └── UObjectMixerCreatorFilter (ObjectMixerCreator base class)
            └── U[FilterName]Filter  (generated Blueprint)
```

`UObjectMixerCreatorFilter` holds the logic for reading the config DataAsset and forwarding the results to Object Mixer. The generated Blueprint adds nothing to this logic — it only provides a concrete class that Object Mixer can enumerate and the user can select.

## Create vs Update Mode

The panel exposes two modes via a toggle at the top.

### Create New

Use Create New when you want to produce a filter that does not yet exist. You specify a name, select classes, choose properties, and click Create. Both the Blueprint and the config DataAsset are written to disk and the Content Browser syncs to them.

### Update Existing

Use Update Existing when a filter already exists and you want to change its classes or property columns. You select the filter from a dropdown that is automatically populated from the output folder. The panel loads the current config DataAsset, lets you modify the class list and property selections, and then writes only the config DataAsset when you click Update. The Blueprint is not touched and does not need to recompile.

!!! note
    The Update mode dropdown lists filters found at the configured output path. If you move a filter to a different folder, it will not appear in the dropdown until you update the output path in Project Settings to match.

## Property Discovery

When you add a class — either from viewport selection or the class picker — the panel uses Unreal's property reflection system to enumerate all `FProperty` instances on that class. Each property is evaluated against the following criteria before being shown:

- It must have the `CPF_Edit` flag (i.e., it is editable in the Details panel) unless `bIncludeEditConstProperties` is enabled in settings, in which case `CPF_EditConst` properties are also shown
- Inherited properties from parent classes are included only if `bIncludeSuperclassProperties` is enabled; by default the panel shows only properties declared directly on the selected class
- Properties are grouped by their `Category` metadata tag; if a property has no category, it appears under "Uncategorized"

The search box performs a case-insensitive substring match against the property display name. Matching is live — the tree updates as you type. The "Check All" and "Uncheck All" buttons operate on whichever properties are currently visible after filtering, not the full list.

## Configuration

Object Mixer Creator exposes its settings under Project Settings > Plugins > Object Mixer Creator.

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `OutputPath` | Package path string | `/Game/ObjectMixerFilters` | Folder where generated Blueprints and DataAssets are saved. Also the folder scanned for the Update Existing dropdown. |
| `DefaultFilterName` | `FString` | `"MyFilter"` | Pre-filled value in the filter name field when opening Create New mode. |
| `BaseClassRestriction` | `TSubclassOf<AActor>` | `AActor` | Limits the class picker to subclasses of this class. Set to `ALight` to restrict the picker to lights only. Has no effect on classes added from viewport selection. |
| `bIncludeSuperclassProperties` | `bool` | `false` | When true, the property tree also shows properties inherited from parent classes of the selected actor class. |
| `bIncludeEditConstProperties` | `bool` | `false` | When true, read-only `EditConst` properties are included in the property tree alongside editable ones. |
