# Adorn — API Reference

## UAdornComponent

`#include "AdornComponent.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `Equip` | `bool Equip(UAdornItemData* Item)` | Equip item to its designated slot |
| `UnequipSlot` | `void UnequipSlot(EAdornSlot Slot)` | Remove item from slot |
| `UnequipAll` | `void UnequipAll()` | Remove all equipped items |
| `GetEquippedItem` | `UAdornItemData* GetEquippedItem(EAdornSlot Slot)` | Query slot item |
| `GetStat` | `float GetStat(FName StatName)` | Aggregated stat value |
| `GetAllStats` | `TMap<FName, float> GetAllStats()` | All aggregated stats |
| `GetTotalWeight` | `float GetTotalWeight()` | Sum of all item weights |
| `GetActiveSetBonuses` | `TArray<FAdornSetBonus> GetActiveSetBonuses()` | Currently active bonuses |
| `GetPieceCountForSet` | `int32 GetPieceCountForSet(FName SetID)` | Items worn from a given set |
| `IsSlotFilled` | `bool IsSlotFilled(EAdornSlot Slot)` | Check if slot has an item |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnItemEquipped` | `(EAdornSlot, UAdornItemData*)` | Item equipped |
| `OnItemUnequipped` | `(EAdornSlot, UAdornItemData*)` | Item unequipped |
| `OnSetBonusActivated` | `(FName SetID, int32 Threshold)` | Set bonus threshold reached |
| `OnSetBonusDeactivated` | `(FName SetID, int32 Threshold)` | Set bonus threshold lost |

## UAdornItemData

| Property | Type | Description |
|----------|------|-------------|
| `ItemID` | FName | Unique identifier |
| `DisplayName` | FText | Display name |
| `Slot` | EAdornSlot | Which body slot |
| `SetID` | FName | Set membership (empty = no set) |
| `Stats` | TArray\<FAdornStatEntry\> | Stat name/value pairs |
| `Weight` | float | Item weight |
| `Mesh` | USkeletalMesh* | Visual mesh |
| `AttachSocket` | FName | Socket for mesh attachment |
| `VisualMode` | EAdornVisualMode | Attachment or MeshSwap |

## EAdornSlot

```cpp
UENUM(BlueprintType)
enum class EAdornSlot : uint8
{
    Head, Chest, Legs, Hands, Feet,
    Ring1, Ring2, Neck, Back, Trinket
};
```
