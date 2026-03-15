# Arsenal — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Arsenal"` to your `Build.cs`.

## 2. Create an Item Data Asset

1. In the Content Browser, right-click → **Miscellaneous → Data Asset**
2. Select `UArsenalItemData` as the class
3. Fill in the properties:

| Property | Example Value |
|----------|---------------|
| Item ID | `Weapon_Rifle_AK47` |
| Display Name | `AK-47` |
| Slot Type | `Primary` |
| Mesh | Assign your skeletal mesh |
| Actor Class | `BP_ArsenalRifleActor` (optional) |
| Stats | `Damage: 45`, `Range: 800`, `Weight: 3.2` |

## 3. Add the Component

Add `UArsenalComponent` to your character Blueprint or C++ class:

```cpp
// C++ constructor
ArsenalComponent = CreateDefaultSubobject<UArsenalComponent>(TEXT("Arsenal"));
```

## 4. Equip an Item

```cpp
// Blueprint or C++
ArsenalComponent->EquipItem(MyRifleData, EArsenalSlotType::Primary);
```

The component spawns the visual actor (if `ActorClass` is set), attaches it to the correct socket, and rebuilds the stat cache.

## 5. Query Stats

```cpp
float Damage = ArsenalComponent->GetStat(FName("Damage"));
float TotalWeight = ArsenalComponent->GetStat(FName("Weight"));
```

## 6. Get Active Item

```cpp
UArsenalItemData* Active = ArsenalComponent->GetActiveItem();
if (Active)
{
    UE_LOG(LogTemp, Log, TEXT("Active: %s"), *Active->DisplayName.ToString());
}
```

## 7. Cycle Weapons

```cpp
// Cycle to next slot that has an item
ArsenalComponent->CycleNext();
ArsenalComponent->CyclePrevious();

// Switch directly to a slot
ArsenalComponent->SetActiveSlot(EArsenalSlotType::Secondary);
```

## 8. Responding to Changes (Blueprint)

Bind to the delegates exposed by `UArsenalComponent`:

```
OnItemEquipped    (SlotType, ItemData)
OnItemUnequipped  (SlotType, ItemData)
OnActiveItemChanged (OldItem, NewItem)
```

## 9. Unequip

```cpp
ArsenalComponent->UnequipSlot(EArsenalSlotType::Primary);
ArsenalComponent->UnequipAll();
```
