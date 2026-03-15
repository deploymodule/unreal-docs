# Vault — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Vault** — enable and restart.

## Step 2: Create Item Definitions

1. Content Browser > right-click > **Data Asset > VaultItemDefinition**.
2. Fill in **Display Name**, **Weight**, **MaxStackSize**, **Tags**, and **ItemMesh** (soft reference).

## Step 3: Add UVaultContainerComponent to Your Actor

=== "Blueprint"
    1. Open your Character or Chest Blueprint.
    2. **Add Component > VaultContainerComponent**.
    3. Set **Slot Count**, **Max Weight**, and optionally assign a **Rule Set** asset.

=== "C++"
    ```cpp
    #include "Vault/VaultContainerComponent.h"

    UPROPERTY(VisibleAnywhere)
    TObjectPtr<UVaultContainerComponent> Inventory;

    // Constructor
    Inventory = CreateDefaultSubobject<UVaultContainerComponent>(TEXT("Inventory"));
    Inventory->SlotCount = 20;
    Inventory->MaxWeight = 100.f;
    ```

## Step 4: Add and Remove Items

=== "C++"
    ```cpp
    UVaultSubsystem* Vault = GetGameInstance()->GetSubsystem<UVaultSubsystem>();

    // Add 3 iron swords
    FPrimaryAssetId SwordId(TEXT("Item"), TEXT("Sword_Iron"));
    EVaultResult Result = Vault->AddItem(Inventory, SwordId, 3);

    if (Result == EVaultResult::Success)
    {
        // Item added
    }
    else if (Result == EVaultResult::WeightExceeded)
    {
        ShowEncumberedMessage();
    }

    // Remove 1 sword
    Vault->RemoveItem(Inventory, SwordId, 1);
    ```

=== "Blueprint"
    - Call **Vault > Add Item** with the Container, Item ID, and Count.
    - Check the returned `EVaultResult` enum for success or failure reason.

## Step 5: Hydrate an Item for Inspection

```cpp
Vault->HydrateItem(SwordId, FOnVaultItemHydrated::CreateLambda([this](UVaultItemDefinition* Def)
{
    ShowItemInspectionPanel(Def);
}));
```

## Step 6: Transfer Between Containers

```cpp
// Move all swords from player inventory to chest
Vault->TransferItem(PlayerInventory, ChestInventory, SwordId, /* count */ 3);
```

## Step 7: Query Contents

```cpp
// Get all entries in the container
TArray<FVaultItemEntry> Entries = Inventory->GetAllEntries();

// Get a specific entry
FVaultItemEntry Entry;
bool bFound = Inventory->GetEntry(SwordId, Entry);

// Get total count of an item across all containers owned by this actor
int32 TotalCount = Vault->GetTotalCount(PlayerActor, SwordId);
```
