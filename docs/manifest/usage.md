# Manifest — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Manifest** — enable and restart.

## Step 2: Create a Manifest Catalog

1. Content Browser > right-click > **Data Asset > ManifestCatalog**.
2. Add `FPrimaryAssetId` entries for your definitions (e.g., all item Data Assets).
3. Add metadata tags and categories for filtering.

## Step 3: Register the Catalog in Project Settings

**Project Settings > Manifest > Catalogs** — add your `UManifestCatalog` assets. They are loaded automatically on game startup.

## Step 4: Resolve an Asset at Runtime

=== "C++"
    ```cpp
    #include "Manifest/ManifestSubsystem.h"

    UManifestSubsystem* Manifest = GetGameInstance()->GetSubsystem<UManifestSubsystem>();

    FPrimaryAssetId SwordId = FPrimaryAssetId(TEXT("Item"), TEXT("Sword_Iron"));

    Manifest->Resolve(SwordId, ECourierPriority::High,
        FOnManifestResolved::CreateLambda([this](UManifestResolvedObject* Resolved)
        {
            if (Resolved && Resolved->IsLoaded())
            {
                UItemDefinition* ItemDef = Cast<UItemDefinition>(Resolved->GetObject());
                EquipItem(ItemDef);
            }
        })
    );
    ```

=== "Blueprint"
    - Call **Manifest > Resolve Asset** with a `Primary Asset Id` and priority.
    - Bind the **On Resolved** output to your callback.

## Step 5: GUID Pool Usage

```cpp
FManifestGuidPool& Pool = Manifest->GetGuidPool(TEXT("SpawnedActors"));

// Generate a new deterministic GUID
FGuid NewGuid = Pool.GenerateGuid();

// Validate that a GUID belongs to this pool
bool bValid = Pool.IsValidGuid(SomeGuid);
```

## Step 6: Query the Catalog

```cpp
// Get all registered asset IDs in a category
TArray<FPrimaryAssetId> AllItems = Manifest->GetAssetsByCategory(FName("Item"));

// Get all assets with a specific tag
TArray<FPrimaryAssetId> Weapons = Manifest->GetAssetsByTag(FGameplayTag::RequestGameplayTag("Type.Weapon"));
```

## Step 7: Unregister / Hot-Reload a Catalog

```cpp
// Useful for DLC scenarios
Manifest->RegisterCatalog(DLCCatalogAsset);
Manifest->UnregisterCatalog(DLCCatalogAsset);
```
