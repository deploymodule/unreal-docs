# Adorn — Overview

## Architecture

```
UAdornComponent  (UActorComponent, replicated)
    ├── TMap<EAdornSlot, UAdornItemData*>  EquippedItems
    ├── FAdornStatCache                    AggregatedStats
    ├── TArray<FAdornSetBonus>             ActiveSetBonuses
    └── TArray<UAdornVisualEntry*>         VisualAttachments

UAdornItemData  (UDataAsset)
    ├── FName ItemID
    ├── EAdornSlot Slot
    ├── FName SetID                        — set bonus membership
    ├── TArray<FAdornStatEntry> Stats
    └── USkeletalMesh* Mesh (or UStaticMesh*)

UAdornSetData  (UDataAsset)
    ├── FName SetID
    └── TArray<FAdornSetBonusThreshold>    — "Equip 2: +10% Damage", "Equip 4: +25% Damage"
```

## Slot System

Default slots cover standard RPG body locations:

| Slot | Description |
|------|-------------|
| `Head` | Helmet, hat, crown |
| `Chest` | Armor, shirt |
| `Legs` | Pants, greaves |
| `Hands` | Gloves, gauntlets |
| `Feet` | Boots, shoes |
| `Ring1` / `Ring2` | Finger rings |
| `Neck` | Amulet, necklace |
| `Back` | Cape, wings |
| `Trinket` | Generic accessory |

## Stat Aggregation

Every time an item is equipped or unequipped, Adorn rebuilds `FAdornStatCache` by summing all stats across equipped items, then adding set bonus stats. Stats are identified by `FName` keys — no GAS dependency.

## Set Bonuses

Set bonuses activate when the player wears a threshold number of items from the same `SetID`. Set data is defined in `UAdornSetData` assets:

```
FireKnight Set:
  2 pieces: +15% Fire Resistance
  4 pieces: +30% Fire Damage, Immunity to Burn
```

When equip state changes, Adorn counts pieces per set, evaluates thresholds, and adds/removes set bonus stats from the cache accordingly.

## Visual System

Adorn supports two visual modes:

1. **Mesh Attachment** — Spawns a static or skeletal mesh actor and attaches it to a socket
2. **Mesh Swap** — Replaces a skeletal mesh component's mesh (for full body costume systems)

Visual mode is set per-item in `UAdornItemData`.
