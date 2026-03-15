# Vault — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Vault"` to your `Build.cs` dependencies.

## 2. Enable the Plugin

Open **Edit > Plugins**, search for **Vault**, enable it, and restart the editor.

## 3. Create Item Definitions

=== "Editor Workflow"
    1. Content Browser > right-click > **Miscellaneous > Data Asset > VaultEditorItemDefinition**.
    2. Fill in **Id** (set the PrimaryAssetId, Category tag, and Type tag).
    3. Set **DisplayName**, **Weight**, **MaxStack**, **MaxDurability**, and **Tags**.
    4. Drag-and-drop meshes, textures, sounds into the visual/audio sections.
    5. Use **UVaultItemConverter > ConvertSingle** or **ConvertFolder** to generate runtime `UVaultItemDefinition` data assets with soft references.

=== "Direct Data Asset"
    1. Content Browser > right-click > **Miscellaneous > Data Asset > VaultItemDefinition**.
    2. Configure the `ItemDef` property directly with soft references for all visual/audio data.

!!! tip "Fragments for Custom Data"
    Use the **Fragments** map to attach game-specific data to any item definition. For example, add a "Stats" fragment with an `FInstancedStruct` containing damage, fire rate, and range values. Access it at runtime with `GetFragment<T>(Tag)`.

## 4. Add a Container Component

=== "Blueprint"
    1. Open your Character, Chest, or Vendor Blueprint.
    2. **Add Component > VaultContainerComponent**.
    3. In the Details panel, set:
        - **Mode** — `SlotBased`, `WeightBased`, `SlotAndWeight`, or `Unlimited`
        - **MaxSlots** — maximum item slots (if slot-based)
        - **MaxWeight** — weight capacity (if weight-based)
        - **bStackSimilarItems** — whether identical items auto-stack

=== "C++"
    ```cpp
    #include "VaultContainerComponent.h"

    UPROPERTY(VisibleAnywhere, Category = "Inventory")
    TObjectPtr<UVaultContainerComponent> Inventory;

    // In constructor
    Inventory = CreateDefaultSubobject<UVaultContainerComponent>(TEXT("Inventory"));
    Inventory->Mode = EVaultContainerMode::SlotAndWeight;
    Inventory->MaxSlots = 20;
    Inventory->MaxWeight = 100.f;
    Inventory->bStackSimilarItems = true;
    ```

## 5. Register Item Definitions

Before items can be created, their definitions must be registered with the subsystem.

=== "Blueprint"
    1. On **BeginPlay**, get the **Vault Subsystem** node.
    2. Call **Register Item Definition Asset** and pass your `UVaultItemDefinition` data asset.
    3. Alternatively, call **Register Definition** with an `FVaultItemDef` struct directly.

=== "C++"
    ```cpp
    #include "VaultSubsystem.h"

    UVaultSubsystem* VaultSub = UVaultSubsystem::Get(this);

    // Register from a loaded data asset
    VaultSub->RegisterItemDefinitionAsset(SwordDefinition);

    // Or register a raw definition struct
    VaultSub->RegisterDefinition(MyItemDef);
    ```

## 6. Add and Remove Items

=== "Blueprint"
    1. Get the **Vault Subsystem** and call **Create Item Instance** with the item's `PrimaryAssetId` and quantity.
    2. Call **Add Item** on your `VaultContainerComponent`, passing the created instance.
    3. Check the returned `FVaultTransactionResult`:
        - **bSuccess** — whether the operation succeeded
        - **QuantityProcessed** / **QuantityRejected** — how many items were accepted/rejected
        - **FailReason** — text description if failed
    4. To remove, call **Remove Item** with the `InstanceId` and quantity to remove (0 = remove all).

=== "C++"
    ```cpp
    UVaultSubsystem* VaultSub = UVaultSubsystem::Get(this);

    // Create an item instance
    FPrimaryAssetId SwordPAID(TEXT("VaultItemDefinition"), TEXT("Sword_Iron"));
    FVaultItemInstance SwordInstance = VaultSub->CreateItemInstance(SwordPAID, 3);

    // Add to inventory
    FVaultTransactionResult Result = Inventory->AddItem(SwordInstance);

    if (Result.bSuccess)
    {
        UE_LOG(LogVault, Log, TEXT("Added %d swords"), Result.QuantityProcessed);
    }
    else
    {
        UE_LOG(LogVault, Warning, TEXT("Failed: %s"), *Result.FailReason.ToString());
    }

    // Remove 1 sword by instance ID
    FVaultTransactionResult RemoveResult = Inventory->RemoveItem(SwordInstance.InstanceId, 1);
    ```

## 7. Equip and Unequip Items

=== "Blueprint"
    1. Call **Equip Item** on the container with the item's `InstanceId`.
    2. The item gains the "Equipped" runtime tag.
    3. If an `IVaultEquipmentBridge` is set, it validates the equip request first.
    4. Call **Unequip Item** to reverse.

=== "C++"
    ```cpp
    // Equip
    FVaultTransactionResult EquipResult = Inventory->EquipItem(SwordInstance.InstanceId);

    // Unequip
    FVaultTransactionResult UnequipResult = Inventory->UnequipItem(SwordInstance.InstanceId);
    ```

## 8. Use / Consume Items

=== "Blueprint"
    Call **Use Item** with the `InstanceId` and quantity to consume. The item's quantity decreases; if it reaches zero, the item is removed. The `OnItemUsed` delegate fires.

=== "C++"
    ```cpp
    // Use 1 health potion
    FVaultTransactionResult UseResult = Inventory->UseItem(PotionInstance.InstanceId, 1);
    ```

## 9. Transfer Between Containers

=== "Blueprint"
    Call **Transfer To** on the source container, passing the `InstanceId`, the target `VaultContainerComponent`, and quantity (0 = transfer all).

=== "C++"
    ```cpp
    // Move 5 arrows from player to chest
    FVaultTransactionResult TransferResult = PlayerInventory->TransferTo(
        ArrowInstance.InstanceId,
        ChestInventory,
        5
    );
    ```

## 10. Apply Modifiers

=== "Blueprint"
    1. Create an `FVaultAppliedModifier` struct with the modifier's `PrimaryAssetId`, type tag, and optional instance data.
    2. Call **Apply Modifier** on the container with the item's `InstanceId`.
    3. The modifier is added to the item's modifier list if the item accepts that modifier type and has available slots.

=== "C++"
    ```cpp
    FVaultAppliedModifier FireEnchant;
    FireEnchant.ModifierPAID = FPrimaryAssetId(TEXT("VaultModDefinition"), TEXT("Mod_FireDamage"));
    FireEnchant.ModifierType = FGameplayTag::RequestGameplayTag(TEXT("Mod.Enchant.Fire"));

    FVaultTransactionResult ModResult = Inventory->ApplyModifier(SwordInstance.InstanceId, FireEnchant);
    ```

## 11. Use Rules (Crafting, Salvage, Repair)

=== "Blueprint"
    1. Create a rule asset or instance (e.g., `VaultRule_Craft`) and configure its inputs and outputs.
    2. Call **Can Apply Rule** from the `VaultFunctionLibrary` to check prerequisites.
    3. Call **Apply Rule** to execute.

=== "C++"
    ```cpp
    #include "VaultFunctionLibrary.h"

    // Check if crafting is possible
    if (UVaultFunctionLibrary::CanApplyRule(Inventory, ItemInstanceId, CraftingRule))
    {
        FVaultTransactionResult CraftResult = UVaultFunctionLibrary::ApplyRule(
            Inventory, ItemInstanceId, CraftingRule
        );
    }
    ```

## 12. Query Inventory Contents

=== "Blueprint"
    | Node | Description |
    |------|-------------|
    | **Get All Items** | Returns all `FVaultItemInstance` entries in the container |
    | **Has Item** | Check if a specific `PrimaryAssetId` exists |
    | **Count Item** | Count total quantity of a specific item |
    | **Find By Tag** | Find items matching a gameplay tag |
    | **Find By PAID** | Find all instances of a specific definition |
    | **Get Current Weight** | Total weight in the container |
    | **Is Empty** / **Is Full** | Container state checks |

=== "C++"
    ```cpp
    // Get all items
    TArray<FVaultItemInstance> AllItems = Inventory->GetAllItems();

    // Check for a specific item
    bool bHasSword = Inventory->HasItem(SwordPAID);
    int32 SwordCount = Inventory->CountItem(SwordPAID);

    // Find items by tag
    FGameplayTag WeaponTag = FGameplayTag::RequestGameplayTag(TEXT("Item.Weapon"));
    TArray<FVaultItemInstance> Weapons = Inventory->FindByTag(WeaponTag);

    // Weight and capacity
    float CurrentWeight = Inventory->GetCurrentWeight();
    bool bFull = Inventory->IsFull();
    ```

## 13. Respond to Inventory Events

=== "Blueprint"
    In your Blueprint, select the `VaultContainerComponent` and bind to events in the Details panel:

    - **On Item Added** — fires when an item is added
    - **On Item Removed** — fires when an item is removed
    - **On Item Equipped** / **On Item Unequipped** — equip state changes
    - **On Item Used** — item consumed
    - **On Item Modified** — modifier applied/removed
    - **On Container Full** — container rejected an add due to capacity
    - **On Transaction Completed** — fires for every operation with the full result

=== "C++"
    ```cpp
    // Bind to delegates
    Inventory->OnItemAdded.AddDynamic(this, &AMyCharacter::HandleItemAdded);
    Inventory->OnItemRemoved.AddDynamic(this, &AMyCharacter::HandleItemRemoved);
    Inventory->OnContainerFull.AddDynamic(this, &AMyCharacter::HandleContainerFull);
    Inventory->OnTransactionCompleted.AddDynamic(this, &AMyCharacter::HandleTransaction);

    void AMyCharacter::HandleItemAdded(FVaultItemInstance Item)
    {
        UpdateInventoryUI();
    }
    ```

## 14. Transaction History and Rollback

=== "Blueprint"
    1. Add a **VaultTransactionComponent** alongside your `VaultContainerComponent`.
    2. Set **MaxHistorySize** in the Details panel.
    3. Call **Rollback Last** to undo the most recent operation, or **Rollback** with a specific `TransactionId`.
    4. Call **Get History** to display a transaction log in your UI.

=== "C++"
    ```cpp
    #include "VaultTransactionComponent.h"

    UPROPERTY(VisibleAnywhere)
    TObjectPtr<UVaultTransactionComponent> TransactionHistory;

    // In constructor
    TransactionHistory = CreateDefaultSubobject<UVaultTransactionComponent>(TEXT("TransactionHistory"));
    TransactionHistory->MaxHistorySize = 50;

    // Undo last operation
    bool bRolledBack = TransactionHistory->RollbackLast();

    // Get full history
    const TArray<FVaultTransaction>& Log = TransactionHistory->GetHistory();
    ```

## 15. Tiered Asset Loading

=== "Blueprint"
    Use the **Vault Function Library** load request preset nodes:

    | Preset | Loads |
    |--------|-------|
    | **Load Request World Only** | World mesh, loot effects |
    | **Load Request Player Pickup** | World + UI icons |
    | **Load Request Player Equip FP** | First-person + audio |
    | **Load Request Player Equip Full** | FP + TP + audio |
    | **Load Request NPC Equip** | Third-person + audio |
    | **Load Request All** | All tiers |

    Pass the request to your resolver or use `GetTierSoftPaths` / `GetTierPAIDs` to get asset references for loading.

=== "C++"
    ```cpp
    #include "VaultFunctionLibrary.h"

    // Get a preset request
    FVaultLoadRequest Request = FVaultLoadRequest::PlayerEquipFP();

    // Get soft paths for a specific tier
    TArray<FSoftObjectPath> FPPaths = UVaultFunctionLibrary::GetTierSoftPaths(ItemDef, EVaultLoadTier::FirstPerson);

    // Check if a tier has data
    bool bHasWorldMesh = UVaultFunctionLibrary::HasTierData(ItemDef, EVaultLoadTier::World);
    ```

## 16. Integration with External Systems

### Resolver (Manifest / Courier Bridge)

=== "Blueprint"
    1. Create a Blueprint implementing `IVaultResolver`.
    2. Override **Resolve Item**, **Is Resolved**, and **Unresolve Item**.
    3. On BeginPlay, call **Set Resolver** on the Vault Subsystem.

=== "C++"
    ```cpp
    // Set up the resolver bridge
    UVaultSubsystem* VaultSub = UVaultSubsystem::Get(this);
    VaultSub->SetResolver(MyManifestResolverObject);
    ```

### Equipment Bridge (Arsenal Integration)

=== "Blueprint"
    1. Create a Blueprint implementing `IVaultEquipmentBridge`.
    2. Override **On Equip Requested** (return true to allow), **On Unequip Requested**, and **On Stats Updated**.
    3. On BeginPlay, call **Set Equipment Bridge** on the Vault Subsystem.

=== "C++"
    ```cpp
    VaultSub->SetEquipmentBridge(MyArsenalBridgeObject);
    ```

### Container Notify Interface

=== "Blueprint"
    Have your actor Blueprint implement `IVaultContainerNotify`. Override **On Vault Item Added**, **On Vault Item Removed**, and **On Vault Container Full** — these fire automatically without binding delegates.

=== "C++"
    ```cpp
    class AMyChest : public AActor, public IVaultContainerNotify
    {
        virtual void OnVaultItemAdded_Implementation(const FVaultItemInstance& Item) override
        {
            PlayLootDepositEffect();
        }
    };
    ```
