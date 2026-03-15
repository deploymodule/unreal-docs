# Adorn — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Adorn"` to your `Build.cs`.

## 2. Create an Item Data Asset

1. Right-click → **Data Asset** → `UAdornItemData`
2. Fill in the properties:

| Property | Example |
|----------|---------|
| Item ID | `Helmet_FireKnight` |
| Display Name | `Fire Knight Helmet` |
| Slot | `Head` |
| Set ID | `FireKnight` |
| Stats | `Defense: 20`, `FireResistance: 15` |
| Mesh | Assign your skeletal mesh |

## 3. Create a Set Data Asset (Optional)

1. Right-click → **Data Asset** → `UAdornSetData`
2. Configure:

| Field | Value |
|-------|-------|
| Set ID | `FireKnight` |
| Thresholds | 2 pieces: `FireResistance +15%`, 4 pieces: `FireDamage +30%` |

## 4. Add the Component

```cpp
AdornComponent = CreateDefaultSubobject<UAdornComponent>(TEXT("Adorn"));
```

## 5. Equip an Item

```cpp
UAdornComponent* Adorn = GetComponentByClass<UAdornComponent>();
Adorn->Equip(FireKnightHelmetData);
```

The component places the item in the correct slot, updates stats, and spawns/swaps visuals automatically.

## 6. Query Stats

```cpp
float Defense     = Adorn->GetStat(FName("Defense"));
float FireRes     = Adorn->GetStat(FName("FireResistance"));
float TotalWeight = Adorn->GetTotalWeight();
```

## 7. Check Active Set Bonuses

```cpp
TArray<FAdornSetBonus> Bonuses = Adorn->GetActiveSetBonuses();
for (const FAdornSetBonus& Bonus : Bonuses)
{
    UE_LOG(LogTemp, Log, TEXT("Set: %s, Pieces: %d"), *Bonus.SetID.ToString(), Bonus.ActivePieceCount);
}
```

## 8. Unequip

```cpp
Adorn->UnequipSlot(EAdornSlot::Head);
Adorn->UnequipAll();
```

## 9. Respond to Events

```cpp
Adorn->OnItemEquipped.AddDynamic(this, &AMyChar::OnItemEquipped);
Adorn->OnSetBonusActivated.AddDynamic(this, &AMyChar::OnSetBonusActivated);
```
