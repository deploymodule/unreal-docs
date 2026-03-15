# Vault — API Reference

## UVaultSubsystem

`#include "VaultSubsystem.h"` — World subsystem managing item definitions, instance IDs, and integration bridges.

### Access

=== "Blueprint"
    Use the **Get Vault Subsystem** node from `VaultFunctionLibrary`.

=== "C++"
    ```cpp
    UVaultSubsystem* VaultSub = UVaultSubsystem::Get(WorldContextObject);
    ```

### Definition Registry

=== "Blueprint"
    | Node | Description |
    |------|-------------|
    | **Register Item Definition Asset** | Register a `UVaultItemDefinition` data asset |
    | **Register Definition** | Register a raw `FVaultItemDef` struct |
    | **Unregister Definition** | Remove a definition by `PrimaryAssetId` |
    | **Find Definition** | Look up a definition by PAID (returns bool + out param) |
    | **Query By Tag** | Find all definitions matching a gameplay tag |
    | **Query By Tag Query** | Find all definitions matching a tag query |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `RegisterDefinition` | `void (const FVaultItemDef&)` | Register a raw definition |
    | `UnregisterDefinition` | `void (const FPrimaryAssetId&)` | Remove a definition |
    | `RegisterItemDefinitionAsset` | `void (UVaultItemDefinition*)` | Register from data asset |
    | `FindDefinition` | `const FVaultItemDef* (const FPrimaryAssetId&)` | Returns nullptr if not found |
    | `K2_FindDefinition` | `bool (const FPrimaryAssetId&, FVaultItemDef& OutDef)` | Blueprint-safe version |
    | `QueryByTag` | `TArray<FVaultItemDef> (const FGameplayTag&)` | Filter by single tag |
    | `QueryByTagQuery` | `TArray<FVaultItemDef> (const FGameplayTagQuery&)` | Filter by tag query |

### Instance Management

=== "Blueprint"
    | Node | Description |
    |------|-------------|
    | **Create Item Instance** | Allocate a unique instance from a definition PAID + quantity |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `AllocateInstanceId` | `int32 ()` | Thread-safe unique ID allocation |
    | `CreateItemInstance` | `FVaultItemInstance (const FPrimaryAssetId&, int32 Quantity)` | Create a fully initialized instance |

### Integration Bridges

=== "Blueprint"
    | Node | Description |
    |------|-------------|
    | **Set Resolver** | Assign an `IVaultResolver` implementation for async item resolution |
    | **Set Equipment Bridge** | Assign an `IVaultEquipmentBridge` for equip/unequip validation |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `SetResolver` | `void (TScriptInterface<IVaultResolver>)` | Set resolver bridge |
    | `GetResolver` | `TScriptInterface<IVaultResolver> ()` | Get current resolver |
    | `SetEquipmentBridge` | `void (TScriptInterface<IVaultEquipmentBridge>)` | Set equipment bridge |
    | `GetEquipmentBridge` | `TScriptInterface<IVaultEquipmentBridge> ()` | Get current bridge |

---

## UVaultContainerComponent

`#include "VaultContainerComponent.h"` — Core inventory container component. Attach to any actor.

### Configuration Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Mode` | `EVaultContainerMode` | `Unlimited` | Container constraint mode |
| `MaxSlots` | `int32` | `0` | Maximum slots (SlotBased/SlotAndWeight modes) |
| `MaxWeight` | `float` | `0.0` | Maximum weight (WeightBased/SlotAndWeight modes) |
| `bUseUniqueInstances` | `bool` | `false` | Create unique instances per item |
| `bStackSimilarItems` | `bool` | `true` | Auto-stack identical items |

### Core Operations

All return `FVaultTransactionResult`.

=== "Blueprint"
    | Node | Description |
    |------|-------------|
    | **Add Item** | Add an `FVaultItemInstance` to the container |
    | **Remove Item** | Remove by `InstanceId` and quantity (0 = all) |
    | **Transfer To** | Move items to another container |
    | **Equip Item** | Mark item as equipped |
    | **Unequip Item** | Mark item as unequipped |
    | **Use Item** | Consume quantity of an item |
    | **Apply Modifier** | Add a modifier to an item |
    | **Remove Modifier** | Remove a modifier by PAID |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `AddItem` | `FVaultTransactionResult (const FVaultItemInstance&)` | Add item to container |
    | `RemoveItem` | `FVaultTransactionResult (int32 InstanceId, int32 Qty)` | Remove by instance ID (0 = all) |
    | `TransferTo` | `FVaultTransactionResult (int32 InstanceId, UVaultContainerComponent*, int32 Qty)` | Transfer to target (0 = all) |
    | `EquipItem` | `FVaultTransactionResult (int32 InstanceId)` | Equip item |
    | `UnequipItem` | `FVaultTransactionResult (int32 InstanceId)` | Unequip item |
    | `UseItem` | `FVaultTransactionResult (int32 InstanceId, int32 Qty)` | Consume item |
    | `ApplyModifier` | `FVaultTransactionResult (int32 InstanceId, const FVaultAppliedModifier&)` | Apply modifier |
    | `RemoveModifier` | `FVaultTransactionResult (int32 InstanceId, const FPrimaryAssetId&)` | Remove modifier |

### Constraint Override

=== "Blueprint"
    Override the **Can Add Item** event in your Blueprint to implement custom container logic. Return `true` to allow, `false` to reject.

=== "C++"
    ```cpp
    // Override in your subclass or use the BlueprintNativeEvent
    virtual bool CanAddItem_Implementation(const FVaultItemInstance& Item) override;
    ```

### Query Functions

=== "Blueprint"
    | Node | Returns | Description |
    |------|---------|-------------|
    | **Has Item** | `bool` | Check if PAID exists in container |
    | **Count Item** | `int32` | Total quantity of a PAID |
    | **Get Item By Instance Id** | `FVaultItemInstance` | Look up specific instance |
    | **Get All Items** | `TArray<FVaultItemInstance>` | All items in container |
    | **Find By Tag** | `TArray<FVaultItemInstance>` | Filter by gameplay tag |
    | **Find By PAID** | `TArray<FVaultItemInstance>` | All instances of a definition |
    | **Get Current Weight** | `float` | Total weight |
    | **Get Item Count** | `int32` | Number of distinct items |
    | **Is Empty** | `bool` | No items in container |
    | **Is Full** | `bool` | Container at capacity |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `HasItem` | `bool (const FPrimaryAssetId&)` | Check existence |
    | `CountItem` | `int32 (const FPrimaryAssetId&)` | Total quantity |
    | `GetItemByInstanceId` | `FVaultItemInstance (int32)` | Look up instance |
    | `GetAllItems` | `TArray<FVaultItemInstance> ()` | All items |
    | `FindByTag` | `TArray<FVaultItemInstance> (const FGameplayTag&)` | Filter by tag |
    | `FindByPAID` | `TArray<FVaultItemInstance> (const FPrimaryAssetId&)` | By definition |
    | `GetCurrentWeight` | `float ()` | Sum weight |
    | `GetItemCount` | `int32 ()` | Distinct item count |
    | `IsEmpty` | `bool ()` | Empty check |
    | `IsFull` | `bool ()` | Capacity check |

### Delegates

| Delegate | Parameter | Description |
|----------|-----------|-------------|
| `OnItemAdded` | `FVaultItemInstance` | Item was added |
| `OnItemRemoved` | `FVaultItemInstance` | Item was removed |
| `OnItemModified` | `FVaultItemInstance` | Modifier applied/removed |
| `OnItemEquipped` | `FVaultItemInstance` | Item equipped |
| `OnItemUnequipped` | `FVaultItemInstance` | Item unequipped |
| `OnItemUsed` | `FVaultItemInstance` | Item consumed |
| `OnContainerFull` | — | Add rejected due to capacity |
| `OnTransactionCompleted` | `FVaultTransactionResult` | Any operation completed |

### Replication

Server RPCs (called automatically when the component is replicated):

| RPC | Parameters | Description |
|-----|-----------|-------------|
| `ServerAddItem` | `FVaultItemInstance` | Server-authoritative add |
| `ServerRemoveItem` | `int32 InstanceId, int32 Qty` | Server-authoritative remove |
| `ServerEquipItem` | `int32 InstanceId` | Server-authoritative equip |
| `ServerUnequipItem` | `int32 InstanceId` | Server-authoritative unequip |

---

## UVaultTransactionComponent

`#include "VaultTransactionComponent.h"` — Optional transaction history and rollback support.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `MaxHistorySize` | `int32` | — | Maximum transactions to keep (ClampMin=1) |

### Functions

=== "Blueprint"
    | Node | Returns | Description |
    |------|---------|-------------|
    | **Rollback** | `bool` | Undo a specific transaction by ID |
    | **Rollback Last** | `bool` | Undo the most recent transaction |
    | **Get History** | `TArray<FVaultTransaction>` | Get full transaction log |
    | **Clear History** | — | Wipe the transaction log |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `RecordTransaction` | `void (const FVaultTransaction&)` | Record a transaction (called internally) |
    | `Rollback` | `bool (int32 TransactionId)` | Undo specific transaction |
    | `RollbackLast` | `bool ()` | Undo most recent |
    | `GetHistory` | `const TArray<FVaultTransaction>& ()` | Get log |
    | `ClearHistory` | `void ()` | Wipe log |

### Delegates

| Delegate | Parameter | Description |
|----------|-----------|-------------|
| `OnTransactionRolledBack` | `FVaultTransaction` | Fired after a successful rollback |

### Server RPC

| RPC | Parameters | Description |
|-----|-----------|-------------|
| `ServerRequestRollback` | `int32 TransactionId` | Server-authoritative rollback |

---

## UVaultItemDefinition

`#include "VaultItemDefinition.h"` — Primary data asset for item definitions. Inherits from `UPrimaryDataAsset`.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `ItemDef` | `FVaultItemDef` | The complete item definition (see below) |

---

## UVaultModDefinition

`#include "VaultModDefinition.h"` — Primary data asset for modifier definitions. Inherits from `UPrimaryDataAsset`.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `ModifierPAID` | `FPrimaryAssetId` | Unique modifier identifier |
| `DisplayName` | `FText` | Human-readable name |
| `Description` | `FText` | Description for UI |
| `ModifierType` | `FGameplayTag` | Category tag (e.g., `Mod.Enchant.Fire`) |
| `Tags` | `FGameplayTagContainer` | Tags for filtering |
| `DefaultInstanceData` | `FInstancedStruct` | Default data when attached |
| `Fragments` | `TMap<FGameplayTag, FInstancedStruct>` | Extensible metadata |

---

## UVaultUpgradeDefinition

`#include "VaultUpgradeDefinition.h"` — Bundles modifiers and a rule into an upgrade package. Inherits from `UPrimaryDataAsset`.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `UpgradePAID` | `FPrimaryAssetId` | Unique upgrade identifier |
| `DisplayName` | `FText` | Human-readable name |
| `Description` | `FText` | Description for UI |
| `UpgradeTags` | `FGameplayTagContainer` | Tags for filtering (e.g., `Upgrade.Tier2`) |
| `ModifierPAIDs` | `TArray<FPrimaryAssetId>` | Modifiers to apply |
| `ApplyRule` | `UVaultRule*` | Rule to execute when applying (instanced) |

---

## UVaultRule

`#include "VaultRule.h"` — Abstract base class for inventory rules. **Blueprintable** — create custom rules by subclassing.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `RuleName` | `FText` | Human-readable name |
| `RuleDescription` | `FText` | Description for UI |
| `RuleTags` | `FGameplayTagContainer` | Tags for filtering |

### Functions (Override in Blueprint or C++)

=== "Blueprint"
    | Event | Returns | Description |
    |-------|---------|-------------|
    | **Can Apply** | `bool` | Check if rule prerequisites are met |
    | **Apply** | `FVaultTransactionResult` | Execute the rule |
    | **Get Requirements** | `TArray<FVaultRuleRequirement>` | List what the rule needs (for UI) |
    | **Get Results** | `TArray<FVaultRuleOutput>` | List what the rule produces (for UI) |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `CanApply` | `bool (UVaultContainerComponent*, int32 InstanceId)` | BlueprintNativeEvent |
    | `Apply` | `FVaultTransactionResult (UVaultContainerComponent*, int32 InstanceId)` | BlueprintNativeEvent |
    | `GetRequirements` | `TArray<FVaultRuleRequirement> ()` | BlueprintNativeEvent |
    | `GetResults` | `TArray<FVaultRuleOutput> ()` | BlueprintNativeEvent |

### Built-in Rules

#### UVaultRule_Craft

Crafting recipe — consume input items, produce output items.

| Property | Type | Description |
|----------|------|-------------|
| `Inputs` | `TArray<FVaultRuleRequirement>` | Required input items (consumed if `bConsumed` is true) |
| `Outputs` | `TArray<FVaultRuleOutput>` | Items produced on success |

#### UVaultRule_Salvage

Break down items into component materials.

| Property | Type | Description |
|----------|------|-------------|
| `SalvageableTag` | `FGameplayTag` | Items matching this tag can be salvaged |
| `SalvageOutputs` | `TArray<FVaultRuleOutput>` | Materials produced |

#### UVaultRule_Repair

Restore item durability using materials.

| Property | Type | Description |
|----------|------|-------------|
| `RepairPercent` | `float` | Durability to restore (0.0–1.0) |
| `RepairMaterials` | `TArray<FVaultRuleRequirement>` | Materials consumed |

#### UVaultRule_ApplyModifier

Apply a modifier to an item, consuming required materials.

| Property | Type | Description |
|----------|------|-------------|
| `Modifier` | `FVaultAppliedModifier` | The modifier to apply |
| `MaterialRequirements` | `TArray<FVaultRuleRequirement>` | Materials consumed |

---

## UVaultFunctionLibrary

`#include "VaultFunctionLibrary.h"` — Static Blueprint function library.

### Subsystem Access

=== "Blueprint"
    | Node | Returns | Description |
    |------|---------|-------------|
    | **Get Vault Subsystem** | `UVaultSubsystem*` | Get the world subsystem |
    | **Get Container Component** | `UVaultContainerComponent*` | Get first container on an actor |

=== "C++"
    | Function | Signature |
    |----------|-----------|
    | `GetVaultSubsystem` | `static UVaultSubsystem* (const UObject*)` |
    | `GetContainerComponent` | `static UVaultContainerComponent* (AActor*)` |

### Item Helpers

=== "Blueprint"
    | Node | Returns | Description |
    |------|---------|-------------|
    | **Create Item Instance** | `FVaultItemInstance` | Create instance from PAID + quantity |
    | **Find Definition** | `bool` + out `FVaultItemDef` | Look up a definition |

=== "C++"
    | Function | Signature |
    |----------|-----------|
    | `CreateItemInstance` | `static FVaultItemInstance (const UObject*, const FPrimaryAssetId&, int32)` |
    | `FindDefinition` | `static bool (const UObject*, const FPrimaryAssetId&, FVaultItemDef&)` |

### Rule Helpers

=== "Blueprint"
    | Node | Returns | Description |
    |------|---------|-------------|
    | **Apply Rule** | `FVaultTransactionResult` | Execute a rule on an item |
    | **Can Apply Rule** | `bool` | Check if a rule can be applied |

=== "C++"
    | Function | Signature |
    |----------|-----------|
    | `ApplyRule` | `static FVaultTransactionResult (UVaultContainerComponent*, int32, UVaultRule*)` |
    | `CanApplyRule` | `static bool (UVaultContainerComponent*, int32, UVaultRule*)` |

### Tiered Loading

=== "Blueprint"
    | Node | Returns | Description |
    |------|---------|-------------|
    | **Get Tier Tag** | `FGameplayTag` | Get the tag for a load tier enum |
    | **Has Tier Data** | `bool` | Check if a definition has data for a tier |
    | **Get Tier PAIDs** | `TArray<FPrimaryAssetId>` | Get PAIDs for a tier's assets |
    | **Get Tier Soft Paths** | `TArray<FSoftObjectPath>` | Get soft paths for a tier's assets |
    | **Load Request World Only** | `FVaultLoadRequest` | Preset: world mesh only |
    | **Load Request Player Pickup** | `FVaultLoadRequest` | Preset: world + UI |
    | **Load Request Player Equip FP** | `FVaultLoadRequest` | Preset: FP + audio |
    | **Load Request Player Equip Full** | `FVaultLoadRequest` | Preset: FP + TP + audio |
    | **Load Request NPC Equip** | `FVaultLoadRequest` | Preset: TP + audio |
    | **Load Request All** | `FVaultLoadRequest` | Preset: all tiers |

=== "C++"
    | Function | Signature |
    |----------|-----------|
    | `GetTierTag` | `static FGameplayTag (EVaultLoadTier)` |
    | `HasTierData` | `static bool (const FVaultItemDef&, EVaultLoadTier)` |
    | `GetTierPAIDs` | `static TArray<FPrimaryAssetId> (const FVaultItemDef&, EVaultLoadTier)` |
    | `GetTierSoftPaths` | `static TArray<FSoftObjectPath> (const FVaultItemDef&, EVaultLoadTier)` |

### Diagnostics

| Function | Returns | Description |
|----------|---------|-------------|
| `GetParameterInfo` | `FString` | Debug info string |

---

## Interfaces

### IVaultResolver

`#include "VaultInterfaces.h"` — Async item resolution bridge. Implement in Blueprint or C++ to connect Vault to your asset loading system (e.g., Manifest + Courier).

=== "Blueprint"
    Override these events in your implementing Blueprint:

    | Event | Description |
    |-------|-------------|
    | **Resolve Item** | Load assets and create runtime object for an item instance. Call `OnResolved` when done. |
    | **Is Resolved** | Return whether an instance ID is currently resolved |
    | **Unresolve Item** | Release resolved item — unload assets and destroy runtime object |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `ResolveItem` | `void (const FVaultItemInstance&, const FOnVaultResolved&)` | Async resolution |
    | `IsResolved` | `bool (int32 InstanceId)` | Check resolution state |
    | `UnresolveItem` | `void (int32 InstanceId)` | Release resolved item |

**Delegate**: `FOnVaultResolved(int32 InstanceId, bool bSuccess)`

### IVaultEquipmentBridge

`#include "VaultInterfaces.h"` — Equipment system integration. Implement to validate equip requests and sync stats with an external system like Arsenal.

=== "Blueprint"
    | Event | Returns | Description |
    |-------|---------|-------------|
    | **On Equip Requested** | `bool` | Return true to allow equip, false to reject |
    | **On Unequip Requested** | — | Called when item is being unequipped |
    | **On Stats Updated** | — | Called when equipment stats change |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `OnEquipRequested` | `bool (const FVaultItemInstance&, const FVaultItemDef&)` | Validate equip |
    | `OnUnequipRequested` | `void (int32 InstanceId)` | Handle unequip |
    | `OnStatsUpdated` | `void (int32 InstanceId, const FInstancedStruct&)` | Sync stats |

### IVaultContainerNotify

`#include "VaultInterfaces.h"` — Optional notification interface. Implement on your actor to receive events without binding delegates.

=== "Blueprint"
    | Event | Description |
    |-------|-------------|
    | **On Vault Item Added** | Called after item successfully added |
    | **On Vault Item Removed** | Called after item successfully removed |
    | **On Vault Container Full** | Called when container rejects an add |

=== "C++"
    | Function | Signature |
    |----------|-----------|
    | `OnVaultItemAdded` | `void (const FVaultItemInstance&)` |
    | `OnVaultItemRemoved` | `void (const FVaultItemInstance&)` |
    | `OnVaultContainerFull` | `void ()` |

---

## Data Structures

### FVaultItemId

Identity struct for item definitions.

| Field | Type | Description |
|-------|------|-------------|
| `PAID` | `FPrimaryAssetId` | Primary asset identifier |
| `Category` | `FGameplayTag` | Broad category (e.g., `Item.Weapon`) |
| `Type` | `FGameplayTag` | Specific type (e.g., `Weapon.Rifle`) |

### FVaultItemDef

Complete item definition data.

| Field | Type | Description |
|-------|------|-------------|
| `Id` | `FVaultItemId` | Identity (PAID, category, type) |
| `DisplayName` | `FText` | Human-readable name |
| `Description` | `FText` | Longer description |
| `Weight` | `float` | Weight per unit (ClampMin=0) |
| `MaxStack` | `int32` | Max stack size (1=unstackable, 0=unlimited) |
| `MaxDurability` | `float` | Max durability (0=indestructible) |
| `Tags` | `FGameplayTagContainer` | Tags for filtering and rule matching |
| `Fragments` | `TMap<FGameplayTag, FInstancedStruct>` | Extensible game-defined data blocks |
| `AcceptedModifierTypes` | `FGameplayTagQuery` | Which modifier types this item accepts |
| `MaxModifierSlots` | `int32` | Max modifier slots (-1=unlimited, 0=none) |

**Template Methods**: `GetFragment<T>(Tag)`, `GetMutableFragment<T>(Tag)`, `SetFragment<T>(Tag, Value)`

### FVaultItemInstance

Runtime item instance in a container.

| Field | Type | Description |
|-------|------|-------------|
| `InstanceId` | `int32` | Unique instance ID (read-only) |
| `DefPAID` | `FPrimaryAssetId` | Which definition this came from |
| `Quantity` | `int32` | Current stack count (ClampMin=1) |
| `Durability` | `float` | Current durability (-1=N/A) |
| `RuntimeTags` | `FGameplayTagContainer` | State tags (Equipped, Locked, etc.) |
| `Modifiers` | `TArray<FVaultAppliedModifier>` | Applied modifiers |
| `InstanceFragments` | `TArray<FVaultInstanceFragment>` | Per-instance data |

**Methods**: `IsValid()`, `HasModifier(PAID)`, `HasModifierOfType(Tag)`, `GetModifiersByType(Tag)`, `AddModifier(Mod, MaxSlots)`, `RemoveModifier(PAID)`, `RemoveModifiersByType(Tag)`

### FVaultAppliedModifier

A modifier applied to an item instance.

| Field | Type | Description |
|-------|------|-------------|
| `ModifierPAID` | `FPrimaryAssetId` | Which modifier definition |
| `ModifierType` | `FGameplayTag` | Category (e.g., `Mod.Scope`) |
| `InstanceData` | `FInstancedStruct` | Per-instance data (roll values, colors, etc.) |

### FVaultSlot

Single slot in a slot-based container.

| Field | Type | Description |
|-------|------|-------------|
| `SlotIndex` | `int32` | Slot index (0-based, read-only) |
| `Item` | `FVaultItemInstance` | Item in this slot |
| `AcceptFilter` | `FGameplayTagQuery` | Optional filter for allowed items |
| `bLocked` | `bool` | Whether slot is locked |

**Methods**: `IsEmpty()`

### FVaultTransaction

Record of a single inventory operation.

| Field | Type | Description |
|-------|------|-------------|
| `TransactionId` | `int32` | Unique transaction ID (read-only) |
| `Type` | `EVaultTransactionType` | Operation type |
| `Before` | `FVaultItemInstance` | Item state before operation |
| `After` | `FVaultItemInstance` | Item state after operation |
| `Timestamp` | `float` | Game time when operation occurred |

### FVaultTransactionResult

Outcome of an inventory operation.

| Field | Type | Description |
|-------|------|-------------|
| `bSuccess` | `bool` | Whether operation succeeded |
| `TransactionId` | `int32` | Transaction ID |
| `QuantityProcessed` | `int32` | Items successfully processed |
| `QuantityRejected` | `int32` | Items rejected |
| `FailReason` | `FText` | Failure description |

### FVaultRuleRequirement

What a rule needs to execute.

| Field | Type | Description |
|-------|------|-------------|
| `RequiredPAID` | `FPrimaryAssetId` | Specific item required (empty=any matching tag) |
| `RequiredTag` | `FGameplayTag` | Tag filter |
| `Quantity` | `int32` | Quantity needed |
| `bConsumed` | `bool` | Whether items are consumed |

### FVaultRuleOutput

What a rule produces.

| Field | Type | Description |
|-------|------|-------------|
| `OutputPAID` | `FPrimaryAssetId` | Item to produce |
| `Quantity` | `int32` | Quantity to produce |

### FVaultWorldItem

Item existing in the world (dropped/placed).

| Field | Type | Description |
|-------|------|-------------|
| `Instance` | `FVaultItemInstance` | The item instance |
| `WorldTransform` | `FTransform` | World location and rotation |
| `TimeInWorld` | `float` | Time since drop |

### FVaultLoadRequest

Tiered loading flags.

| Field | Type | Description |
|-------|------|-------------|
| `bLoadWorld` | `bool` | Load world visuals |
| `bLoadUI` | `bool` | Load UI assets |
| `bLoadFirstPerson` | `bool` | Load FP assets |
| `bLoadThirdPerson` | `bool` | Load TP assets |
| `bLoadAudio` | `bool` | Load audio assets |

**Presets**: `WorldOnly()`, `PlayerPickup()`, `PlayerEquipFP()`, `PlayerEquipFull()`, `NPCEquip()`, `All()`

---

## Visual/Audio Fragments

Item definitions store visual and audio data as fragments, loaded on demand per tier.

### FVaultWorldFragment

Tag: `Vault.Visual.World` — Ground pickup representation.

| Field | Type | Description |
|-------|------|-------------|
| `WorldMesh` | `TSoftObjectPtr<UStaticMesh>` | Pickup static mesh |
| `WorldSkeletalMesh` | `TSoftObjectPtr<USkeletalMesh>` | Animated ground model |
| `WorldMaterialOverride` | `TSoftObjectPtr<UMaterialInterface>` | Material override |
| `WorldEffect` | `TSoftObjectPtr<UObject>` | Loot glow / rarity beam (Niagara) |
| `WorldMeshScale` | `float` | Scale (default=1.0) |
| `WorldMeshRotationOffset` | `FRotator` | Rotation offset |

### FVaultUIFragment

Tag: `Vault.Visual.UI` — Inventory UI representation.

| Field | Type | Description |
|-------|------|-------------|
| `Icon` | `TSoftObjectPtr<UTexture2D>` | Inventory thumbnail |
| `IconLarge` | `TSoftObjectPtr<UTexture2D>` | Detail view icon |
| `ItemCardBackground` | `TSoftObjectPtr<UTexture2D>` | Card art background |
| `IconMaterial` | `TSoftObjectPtr<UMaterialInterface>` | Rarity frame material |

### FVaultFirstPersonFragment

Tag: `Vault.Visual.FP` — First-person representation.

| Field | Type | Description |
|-------|------|-------------|
| `FPMesh` | `TSoftObjectPtr<USkeletalMesh>` | FP arms/weapon mesh |
| `FPAnimBlueprint` | `TSoftObjectPtr<UAnimBlueprint>` | Animation blueprint |
| `FPEquipMontage` | `TSoftObjectPtr<UAnimMontage>` | Equip animation |
| `FPUseMontage` | `TSoftObjectPtr<UAnimMontage>` | Use/fire animation |
| `FPMaterialOverride` | `TSoftObjectPtr<UMaterialInterface>` | Material override |
| `FPMuzzleEffect` | `TSoftObjectPtr<UObject>` | Muzzle flash (Niagara) |
| `FPImpactEffect` | `TSoftObjectPtr<UObject>` | Impact effect (Niagara) |

### FVaultThirdPersonFragment

Tag: `Vault.Visual.TP` — Third-person representation.

| Field | Type | Description |
|-------|------|-------------|
| `TPMesh` | `TSoftObjectPtr<USkeletalMesh>` | TP weapon/item mesh |
| `TPAnimBlueprint` | `TSoftObjectPtr<UAnimBlueprint>` | Animation blueprint |
| `TPEquipMontage` | `TSoftObjectPtr<UAnimMontage>` | Equip animation |
| `TPUseMontage` | `TSoftObjectPtr<UAnimMontage>` | Use/fire animation |
| `TPMaterialOverride` | `TSoftObjectPtr<UMaterialInterface>` | Material override |
| `TPTrailEffect` | `TSoftObjectPtr<UObject>` | Trail/particle effect (Niagara) |

### FVaultAudioFragment

Tag: `Vault.Audio` — Audio assets.

| Field | Type | Description |
|-------|------|-------------|
| `PickupSound` | `TSoftObjectPtr<USoundBase>` | Pickup sound |
| `DropSound` | `TSoftObjectPtr<USoundBase>` | Drop sound |
| `EquipSound` | `TSoftObjectPtr<USoundBase>` | Equip sound |
| `UnequipSound` | `TSoftObjectPtr<USoundBase>` | Unequip sound |
| `UseSound` | `TSoftObjectPtr<USoundBase>` | Use/fire/consume sound |
| `ImpactSound` | `TSoftObjectPtr<USoundBase>` | Impact/hit sound |
| `AmbientSound` | `TSoftObjectPtr<USoundBase>` | Ambient/idle sound |
| `ReloadSound` | `TSoftObjectPtr<USoundBase>` | Reload sound |

---

## Enums

### EVaultContainerMode

| Value | Description |
|-------|-------------|
| `Unlimited` | No constraints |
| `SlotBased` | Limited by slot count |
| `WeightBased` | Limited by total weight |
| `SlotAndWeight` | Both slot and weight limits |
| `Custom` | Override `CanAddItem` for custom logic |

### EVaultTransactionType

| Value | Description |
|-------|-------------|
| `Add` | Item added |
| `Remove` | Item removed |
| `Equip` | Item equipped |
| `Unequip` | Item unequipped |
| `Drop` | Item dropped |
| `Modify` | Modifier applied/removed |
| `Transfer` | Transferred between containers |
| `Use` | Item consumed |
| `RuleApply` | Rule executed |

### EVaultLoadTier

| Value | Tag | Description |
|-------|-----|-------------|
| `World` | `Vault.Visual.World` | Ground pickup assets |
| `UI` | `Vault.Visual.UI` | Inventory UI assets |
| `FirstPerson` | `Vault.Visual.FP` | FP view assets |
| `ThirdPerson` | `Vault.Visual.TP` | TP view assets |
| `Audio` | `Vault.Audio` | Audio assets |

---

## Editor

### UVaultEditorItemDefinition

`#include "VaultEditorItemDefinition.h"` — Editor-only authoring data asset with hard object references for drag-and-drop workflow. Convert to runtime `UVaultItemDefinition` using `UVaultItemConverter`.

Contains the same fields as `FVaultItemDef` plus hard-referenced visual/audio data structs for each tier.

### UVaultItemConverter

`#include "VaultItemConverter.h"` — Utility for converting editor assets to runtime.

=== "Blueprint"
    | Node | Description |
    |------|-------------|
    | **Convert Single** | Convert one `UVaultEditorItemDefinition` to runtime |
    | **Convert Folder** | Convert all editor definitions in a folder (optional: recursive, only dirty) |

=== "C++"
    | Function | Signature | Description |
    |----------|-----------|-------------|
    | `ConvertSingle` | `static FVaultConvertResult (UVaultEditorItemDefinition*)` | Convert one asset |
    | `ConvertFolder` | `static TArray<FVaultConvertResult> (const FString&, bool bRecursive, bool bOnlyDirty)` | Batch convert |
