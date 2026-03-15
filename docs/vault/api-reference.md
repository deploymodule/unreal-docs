# Vault — API Reference

## UVaultSubsystem

| Function | Signature | Description |
|---|---|---|
| `AddItem` | `EVaultResult (UVaultContainerComponent*, FPrimaryAssetId, int32 Count)` | Adds items to a container after running rule checks. |
| `RemoveItem` | `EVaultResult (UVaultContainerComponent*, FPrimaryAssetId, int32 Count)` | Removes items from a container after running rule checks. |
| `TransferItem` | `EVaultResult (UVaultContainerComponent* From, UVaultContainerComponent* To, FPrimaryAssetId, int32 Count)` | Transfers items between containers, validating both containers' rules. |
| `HydrateItem` | `void (FPrimaryAssetId, FOnVaultItemHydrated)` | Asynchronously loads a `UVaultItemDefinition` via Courier. |
| `GetTotalCount` | `int32 (AActor*, FPrimaryAssetId)` | Returns the sum count across all `UVaultContainerComponent`s on the actor. |
| `GetEffectiveStats` | `FVaultItemStats (FVaultItemEntry&)` | Computes final item stats by applying all modifiers in the entry's modifier stack. |
| `GetTransactionLog` | `TArray<FVaultTransaction> ()` | Returns all logged transactions this session. |

## UVaultContainerComponent

| Function / Property | Type | Description |
|---|---|---|
| `SlotCount` | `int32` | Maximum distinct item types this container can hold. |
| `MaxWeight` | `float` | Maximum total weight. |
| `RuleSet` | `UVaultRuleSet*` | Optional rule set Data Asset for add/remove/transfer validation. |
| `GetEntry` | `bool (FPrimaryAssetId, FVaultItemEntry& Out)` | Gets the entry for a specific item. Returns false if not present. |
| `GetAllEntries` | `TArray<FVaultItemEntry> ()` | Returns all item entries in the container. |
| `GetCurrentWeight` | `float ()` | Returns the sum weight of all entries. |
| `GetSlotUsed` | `int32 ()` | Returns the number of distinct item types currently stored. |
| `IsEmpty` | `bool ()` | Returns true if the container has no entries. |
| `OnItemAdded` | Delegate (`FPrimaryAssetId, int32`) | Fired when items are successfully added. |
| `OnItemRemoved` | Delegate (`FPrimaryAssetId, int32`) | Fired when items are successfully removed. |

## UVaultItemDefinition

| Property | Type | Description |
|---|---|---|
| `DisplayName` | `FText` | Human-readable item name. |
| `Weight` | `float` | Per-unit weight. |
| `MaxStackSize` | `int32` | Maximum units per stack slot. |
| `Tags` | `FGameplayTagContainer` | Tags for filtering and rule evaluation. |
| `ItemMesh` | `TSoftObjectPtr<UStaticMesh>` | Soft reference to the pickup/world mesh. |

## FVaultItemEntry (Struct)

| Field | Type | Description |
|---|---|---|
| `ItemId` | `FPrimaryAssetId` | Identifies which item definition this entry references. |
| `Count` | `int32` | Number of items in this stack. |
| `Modifiers` | `TArray<FVaultModifier>` | Applied modifiers (enchantments, conditions, buffs). |
| `Metadata` | `TMap<FName, FString>` | Free-form key-value metadata (e.g., custom roll values). |

## EVaultResult

| Value | Description |
|---|---|
| `Success` | Operation completed successfully. |
| `SlotsFull` | No available slots in the container. |
| `WeightExceeded` | Adding would exceed the weight limit. |
| `StackFull` | Stack limit for this item type reached. |
| `RuleVeto` | A rule set condition blocked the operation. |
| `ItemNotFound` | Requested item not present in the container. |
