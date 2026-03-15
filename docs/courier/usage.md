# Courier — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Courier** — enable and restart.

## Step 2: Request an Asset Load

=== "C++"
    ```cpp
    #include "Courier/CourierSubsystem.h"

    UCourierSubsystem* Courier = GetGameInstance()->GetSubsystem<UCourierSubsystem>();

    FCourierHandle Handle = Courier->RequestLoad(
        WeaponSoftRef,                  // TSoftObjectPtr<UObject> or FSoftObjectPath
        ECourierPriority::High,
        FOnCourierLoaded::CreateLambda([this](UObject* LoadedAsset)
        {
            WeaponMesh = Cast<USkeletalMesh>(LoadedAsset);
            SpawnWeapon();
        })
    );
    ```

=== "Blueprint"
    - Call **Courier > Request Load Async** with a Soft Object Reference and Priority.
    - Bind the **On Loaded** output pin to your callback event.

## Step 3: Keep the Handle Alive

Store the `FCourierHandle` as a member variable for as long as you need the asset loaded:

```cpp
// In your Actor header
FCourierHandle WeaponHandle;

// Assign in BeginPlay or on equip
WeaponHandle = Courier->RequestLoad(WeaponSoftRef, ECourierPriority::High, OnLoaded);

// Release when unequipping — asset unloads when no other handles exist
WeaponHandle.Release();
```

## Step 4: Query Load State

```cpp
ECourierLoadState State = Courier->GetLoadState(WeaponSoftRef);
switch (State)
{
    case ECourierLoadState::Loaded:
        // Safe to use
        break;
    case ECourierLoadState::Loading:
        // Still in flight
        break;
    case ECourierLoadState::Failed:
        UE_LOG(LogGame, Error, TEXT("Asset failed to load."));
        break;
}
```

## Step 5: Batch Loads

```cpp
TArray<FSoftObjectPath> AssetPaths = { MeshPath, TexturePath, MaterialPath };

FCourierBatchHandle BatchHandle = Courier->RequestBatchLoad(
    AssetPaths,
    ECourierPriority::Normal,
    FOnCourierBatchLoaded::CreateLambda([this]()
    {
        InitializeCharacterVisuals();
    })
);
```

## Step 6: Preload by Tag

Courier integrates with the Asset Manager. Tag a group of assets with a `GameplayTag` in their Primary Asset Type config, then:

```cpp
Courier->PreloadByTag(FGameplayTag::RequestGameplayTag("Zone.Forest"), ECourierPriority::Low);
```
