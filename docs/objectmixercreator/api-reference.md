# Object Mixer Creator — API Reference

All classes are declared in the `ObjectMixerCreator` module. Headers are located under `ObjectMixerCreator/Public/`.

---

## UObjectMixerCreatorFilter (Base Class)

`ObjectMixerCreator/Public/ObjectMixerCreatorFilter.h`

Abstract base class that all generated filter Blueprints inherit from. Handles reading the config DataAsset and forwarding the results to Object Mixer via the standard filter interface.

You do not instantiate this class directly. Generated Blueprints subclass it automatically. You may also subclass it manually in C++ if you need custom filter logic beyond what the config DataAsset provides — see [Extending the Base Filter](#extending-the-base-filter).

### Properties

| Property | Type | Access | Description |
|----------|------|--------|-------------|
| `FilterConfig` | `TObjectPtr<UObjectMixerFilterConfig>` | `EditDefaultsOnly` | The config DataAsset instance this filter reads from. Set automatically when the Blueprint is generated. Assign a different config asset to redirect the filter to a different column and class configuration without changing the Blueprint class. |

### Overridden Functions

These functions are declared on `UObjectMixerObjectFilter` (the Unreal built-in base) and overridden here to delegate to the config DataAsset.

| Function | Return Type | Description |
|----------|-------------|-------------|
| `GetObjectClassesToFilter` | `TSet<UClass*>` | Resolves and returns the `ClassesToShow` soft class pointers from `FilterConfig`. Called by Object Mixer when it needs to know which actor classes to include in its list. |
| `GetColumnsToShowByDefault` | `TSet<FName>` | Returns the `Columns` array from `FilterConfig` as a set. Called by Object Mixer when it constructs its column headers. |

!!! note
    Both functions are called by Object Mixer at display time, not at asset creation time. Changes to `FilterConfig` are reflected immediately in the Object Mixer panel the next time it refreshes its view.

---

## UObjectMixerFilterConfig (DataAsset)

`ObjectMixerCreator/Public/ObjectMixerFilterConfig.h`

Plain data asset that stores the class and property column configuration for a filter. Serialized to disk as a standard UE DataAsset. Can be edited in the Details panel independently of Object Mixer Creator.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `ClassesToShow` | `TArray<TSoftClassPtr<AActor>>` | Actor classes the filter passes to Object Mixer's class query. Stored as soft class pointers so they do not force-load Blueprint assets at editor startup. Resolved to hard `UClass*` pointers lazily when `GetObjectClassesToFilter` is called. |
| `Columns` | `TArray<FName>` | Property names to show as columns in Object Mixer. Each entry corresponds to the internal `FName` of an `FProperty` on one of the listed classes. Names are stored verbatim — they must match the property's declared name exactly for Object Mixer to find the correct column. |

!!! tip
    If you need to add or remove columns programmatically (for example, in an editor utility script), you can load the DataAsset, modify `Columns` directly, and call `MarkPackageDirty()` followed by `UEditorAssetLibrary::SaveAsset`. Object Mixer Creator will reflect those changes the next time the Update Existing panel loads the filter.

---

## FObjectMixerFilterFactory

`ObjectMixerCreator/Private/ObjectMixerFilterFactory.h`

Internal factory class that handles all asset creation, update, and deletion operations. Called by `SObjectMixerCreatorPanel` in response to user actions. Not exposed as a public API but documented here for reference when extending or debugging the plugin.

### CreateFilter

```cpp
static bool CreateFilter(
    const FString&                      FilterName,
    const TArray<UClass*>&              Classes,
    const TArray<FName>&                Columns,
    const FString&                      OutputPath
);
```

| Parameter | Description |
|-----------|-------------|
| `FilterName` | Base name for both generated assets. Produces `U[FilterName]Filter` (Blueprint) and `[FilterName]Config` (DataAsset). |
| `Classes` | Array of actor class pointers to store in the config DataAsset's `ClassesToShow`. |
| `Columns` | Array of property `FName` values to store in the config DataAsset's `Columns`. |
| `OutputPath` | Package path where both assets are created. Taken from `UObjectMixerCreatorSettings::OutputPath`. |

Returns `true` if both assets were created and saved successfully. Returns `false` and logs an error if a naming conflict exists or the output directory is not writable. On success, synchronizes the Content Browser to both new assets.

### UpdateFilter

```cpp
static bool UpdateFilter(
    UObjectMixerFilterConfig*           ExistingConfig,
    const TArray<UClass*>&              Classes,
    const TArray<FName>&                Columns
);
```

| Parameter | Description |
|-----------|-------------|
| `ExistingConfig` | Pointer to the config DataAsset that should be overwritten. Obtained from the filter selected in the Update Existing dropdown. |
| `Classes` | New class array to write into `ExistingConfig->ClassesToShow`. Replaces the previous value entirely. |
| `Columns` | New column array to write into `ExistingConfig->Columns`. Replaces the previous value entirely. |

Returns `true` if the DataAsset was saved. The Blueprint is not modified. The Content Browser is not re-synced (the asset already exists in its existing location).

### GetExistingFilters

```cpp
static TArray<FAssetData> GetExistingFilters(const FString& OutputPath);
```

| Parameter | Description |
|-----------|-------------|
| `OutputPath` | Package path to scan. Taken from `UObjectMixerCreatorSettings::OutputPath`. |

Queries the Asset Registry for all Blueprint assets at `OutputPath` whose parent class is `UObjectMixerCreatorFilter`. Returns the results as an array of `FAssetData`. Used by the Update Existing dropdown to populate its options list.

---

## UObjectMixerCreatorSettings

`ObjectMixerCreator/Public/ObjectMixerCreatorSettings.h`

Project Settings class exposed under Edit > Project Settings > Plugins > Object Mixer Creator. Inherits from `UDeveloperSettings`.

```cpp
UCLASS(Config=EditorPerProjectUserSettings, defaultconfig, meta=(DisplayName="Object Mixer Creator"))
class OBJECTMIXERCREATOR_API UObjectMixerCreatorSettings : public UDeveloperSettings
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `OutputPath` | `FString` | `"/Game/ObjectMixerFilters"` | Package path where filter Blueprints and config DataAssets are written. Also used as the scan directory for `GetExistingFilters`. Must be a valid `/Game/` or `/[PluginName]/` content path. |
| `DefaultFilterName` | `FString` | `"MyFilter"` | Pre-filled value in the Filter Name field in Create New mode. Does not affect asset naming in Update Existing mode. |
| `BaseClassRestriction` | `TSubclassOf<AActor>` | `AActor::StaticClass()` | Restricts the class picker dialog to subclasses of this class. Has no effect on classes added via "Add from Selection". |
| `bIncludeSuperclassProperties` | `bool` | `false` | When `true`, property reflection walks the full class hierarchy from the selected class up to (but not including) `UObject`. When `false`, only properties declared directly on the selected class are enumerated. |
| `bIncludeEditConstProperties` | `bool` | `false` | When `true`, properties with `CPF_EditConst` are included in the property tree alongside fully editable properties. When `false`, only `CPF_Edit` properties are shown. |

### Accessing Settings at Runtime

```cpp
#include "ObjectMixerCreatorSettings.h"

const UObjectMixerCreatorSettings* Settings = GetDefault<UObjectMixerCreatorSettings>();
FString OutputPath = Settings->OutputPath;
```

Settings are stored in `EditorPerProjectUserSettings` so they are local to the project and not checked into source control by default.

---

## Extending the Base Filter

If you need filter behavior that the config DataAsset cannot provide — for example, a filter that dynamically includes actors based on a custom tag or a runtime condition — you can subclass `UObjectMixerCreatorFilter` directly in C++.

```cpp
#include "ObjectMixerCreatorFilter.h"

UCLASS()
class MYPLUGIN_API UMyCustomFilter : public UObjectMixerCreatorFilter
{
    GENERATED_BODY()

public:
    // Override to add custom class logic on top of the config DA results
    virtual TSet<UClass*> GetObjectClassesToFilter() const override;

    // Override to compute columns dynamically
    virtual TSet<FName> GetColumnsToShowByDefault() const override;
};
```

```cpp
TSet<UClass*> UMyCustomFilter::GetObjectClassesToFilter() const
{
    // Start with whatever the config DataAsset specifies
    TSet<UClass*> Classes = Super::GetObjectClassesToFilter();

    // Add a class programmatically
    Classes.Add(AMySpecialActor::StaticClass());

    return Classes;
}
```

!!! note
    A manually authored C++ subclass is a standard Unreal Blueprint-compatible class and will appear in the Object Mixer filter dropdown alongside filters generated by Object Mixer Creator. You do not need to run Object Mixer Creator to register it — Object Mixer discovers filter classes through the Asset Registry automatically.

!!! warning
    If you assign a `FilterConfig` DataAsset to a manually-authored subclass, the `Super::` calls will read from that asset. If you leave `FilterConfig` null, the base-class implementations return empty sets, and your overrides are solely responsible for providing the class and column lists.
