# Arsenal — Overview

## Architecture

```
UArsenalComponent  (UActorComponent, replicated)
    ├── TArray<FArsenalSlot>          — Ordered slot array (Primary, Secondary, Melee, etc.)
    ├── UArsenalItemData* ActiveItem  — Currently equipped/active item
    └── FArsenalStatCache             — Aggregated stats from all equipped items

UArsenalItemData  (UDataAsset)
    ├── FName ItemID
    ├── EArsenalSlotType SlotType
    ├── TArray<FArsenalStatEntry> Stats
    ├── TSoftObjectPtr<USkeletalMesh> Mesh
    └── TSubclassOf<AArsenalItemActor> ActorClass
```

## Data Asset Pattern

All item data lives in `UArsenalItemData` assets. These are **hard references** — they are always loaded when the character is loaded. This means:

- Zero latency equip: no async load needed at equip time
- Predictable memory: item data is always in memory
- Simple iteration: `TArray<UArsenalItemData*>` is always safe to iterate

For projects with hundreds of items, organize items into per-level or per-region data sets and only include the relevant data assets in the character's loadout list.

## Slot System

Slots are defined as `EArsenalSlotType` (or extended per-project). Each slot holds one item. The default slots are:

| Slot | Description |
|------|-------------|
| `Primary` | Main weapon (rifle, bow, etc.) |
| `Secondary` | Sidearm or offhand weapon |
| `Melee` | Melee weapon |
| `Gadget` | Utility item |
| `Armor` | Body armor |
| `Helmet` | Head armor |

Projects can add custom slot types by extending `EArsenalSlotType` in their own module.

## Stat Aggregation

When any item is equipped or unequipped, `UArsenalComponent` rebuilds `FArsenalStatCache`. Stats are summed across all equipped items. Stats are identified by `FGameplayTag`-like `FName` keys (no GAS dependency).

Example: if `Helmet` grants `Defense +20` and `Armor` grants `Defense +35`, the cache reports `Defense = 55`.

## Replication

`UArsenalComponent` fully replicates:
- The slot contents (item data pointers replicated by path)
- Active item selection
- Equip/unequip RPCs with authority validation

Clients receive `OnSlotChanged` and `OnActiveItemChanged` delegates.
