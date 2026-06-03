# Object Mixer Creator — Changelog

## v0.1.0 — Initial Release

### Added

- `FObjectMixerCreatorModule` — registers the nomad tab factory and injects the "Object Mixer Filter Creator" entry into the Window menu at editor startup
- `SObjectMixerCreatorPanel` — full Slate UI panel containing the mode toggle, class list, property tree, search box, filter name field, and action buttons (Create, Update, Delete, Add from Selection, Add Class..., Check All, Uncheck All)
- Create New mode — builds a new filter Blueprint and config DataAsset from a user-supplied name, class list, and property selection; syncs the Content Browser to both new assets on completion
- Update Existing mode — populates a dropdown from the Asset Registry scan of the output path; loads an existing config DataAsset into the panel for editing; writes only the DataAsset on Update, leaving the Blueprint untouched
- Delete action in Update Existing mode — removes the filter Blueprint and its config DataAsset together from the Content Browser
- "Add from Selection" button — reads the current viewport selection, enumerates the unique actor classes found, and adds them to the class list in one click
- "Add Class..." button — opens the UE class picker dialog filtered to `AActor` subclasses (or narrower based on `BaseClassRestriction`) for adding a class that is not currently placed in the level
- Per-class expandable rows in the class list with remove buttons; properties grouped by Details panel category within each expanded row
- Live search box performing case-insensitive substring matching against property display names; filters the visible property tree in real time
- "Check All" and "Uncheck All" buttons that operate on the currently visible (post-search) property set
- `UObjectMixerCreatorFilter` — base `UObjectMixerObjectFilter` subclass that all generated Blueprints inherit from; reads `ClassesToShow` and `Columns` from an assigned `UObjectMixerFilterConfig` DataAsset and returns them to Object Mixer via `GetObjectClassesToFilter` and `GetColumnsToShowByDefault`
- `UObjectMixerFilterConfig` — `UDataAsset` subclass with `ClassesToShow` (`TArray<TSoftClassPtr<AActor>>`) and `Columns` (`TArray<FName>`) properties; stored as a plain UE DataAsset independently editable in the Details panel
- `FObjectMixerFilterFactory` — internal factory providing `CreateFilter`, `UpdateFilter`, and `GetExistingFilters`; handles all Asset Registry interactions, Blueprint generation, DataAsset instantiation, and Content Browser sync
- `UObjectMixerCreatorSettings` — `UDeveloperSettings` subclass exposing `OutputPath`, `DefaultFilterName`, `BaseClassRestriction`, `bIncludeSuperclassProperties`, and `bIncludeEditConstProperties` under Project Settings > Plugins > Object Mixer Creator
- Property reflection using `CPF_Edit` flag filtering; optional `CPF_EditConst` inclusion controlled by `bIncludeEditConstProperties`
- Superclass property traversal controlled by `bIncludeSuperclassProperties`; when disabled, only properties declared directly on the selected class are enumerated
- `BaseClassRestriction` setting that narrows the "Add Class..." class picker to a specific `AActor` subclass hierarchy (e.g., `ALight`)
- Generated filter Blueprints are immediately usable in Object Mixer without editor restart; discoverable through Object Mixer's standard filter dropdown via the Asset Registry
- Content Browser sync after Create to focus both generated assets in the browser for immediate inspection
