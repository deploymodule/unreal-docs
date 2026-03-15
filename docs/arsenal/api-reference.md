# Arsenal — API Reference

## UArsenalComponent

`#include "ArsenalComponent.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `EquipItem` | `bool EquipItem(UArsenalItemData* Item, EArsenalSlotType Slot)` | Equip item to a slot |
| `UnequipSlot` | `void UnequipSlot(EArsenalSlotType Slot)` | Remove item from slot |
| `UnequipAll` | `void UnequipAll()` | Clear all slots |
| `GetItemInSlot` | `UArsenalItemData* GetItemInSlot(EArsenalSlotType Slot)` | Query slot contents |
| `GetActiveItem` | `UArsenalItemData* GetActiveItem()` | Currently active item |
| `SetActiveSlot` | `void SetActiveSlot(EArsenalSlotType Slot)` | Switch active slot |
| `CycleNext` | `void CycleNext()` | Cycle to next filled slot |
| `CyclePrevious` | `void CyclePrevious()` | Cycle to previous filled slot |
| `GetStat` | `float GetStat(FName StatName)` | Query aggregated stat value |
| `GetAllStats` | `TMap<FName,float> GetAllStats()` | All aggregated stats |
| `HasItem` | `bool HasItem(UArsenalItemData* Item)` | Check if item is equipped |
| `IsSlotFilled` | `bool IsSlotFilled(EArsenalSlotType Slot)` | Check if slot has an item |
| `GetVisualActor` | `AArsenalItemActor* GetVisualActor(EArsenalSlotType Slot)` | Get spawned visual actor |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnItemEquipped` | `(EArsenalSlotType, UArsenalItemData*)` | Item is equipped to a slot |
| `OnItemUnequipped` | `(EArsenalSlotType, UArsenalItemData*)` | Item is removed from a slot |
| `OnActiveItemChanged` | `(UArsenalItemData* Old, UArsenalItemData* New)` | Active slot changes |

## UArsenalItemData

`#include "ArsenalItemData.h"`

| Property | Type | Description |
|----------|------|-------------|
| `ItemID` | FName | Unique identifier |
| `DisplayName` | FText | Human-readable name |
| `SlotType` | EArsenalSlotType | Which slot this item occupies |
| `Stats` | TArray\<FArsenalStatEntry\> | Stat name/value pairs |
| `Mesh` | USkeletalMesh* | Visual mesh (hard ref) |
| `AttachSocket` | FName | Socket name for attachment |
| `ActorClass` | TSubclassOf\<AArsenalItemActor\> | Spawned visual actor class |
| `EquipSound` | USoundBase* | Played on equip |
| `Icon` | UTexture2D* | UI icon |

## EArsenalSlotType

```cpp
UENUM(BlueprintType)
enum class EArsenalSlotType : uint8
{
    Primary   = 0,
    Secondary = 1,
    Melee     = 2,
    Gadget    = 3,
    Armor     = 4,
    Helmet    = 5,
    Custom    = 255,
};
```
