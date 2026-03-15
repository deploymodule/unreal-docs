# Arsenal — Changelog

## v1.0.0 — Initial Release

### Added
- `UArsenalComponent` — replicated equipment manager with slot-based inventory
- `UArsenalItemData` — hard-reference Data Asset for item definition (mesh, stats, actor class)
- Six default slot types: Primary, Secondary, Melee, Gadget, Armor, Helmet
- Stat aggregation cache rebuilt on every equip/unequip operation
- Visual actor spawning and socket attachment at equip time
- Active item switching with `SetActiveSlot`, `CycleNext`, `CyclePrevious`
- Full replication of slot contents, active item, and equip/unequip RPCs
- Delegates: `OnItemEquipped`, `OnItemUnequipped`, `OnActiveItemChanged`
- Blueprint-callable API for all component functions
- Equip sound playback from `UArsenalItemData.EquipSound`
- GAS-independent stat system using `FName` keys
- Custom slot type extension support via project-level `EArsenalSlotType` extension
