# Adorn — Changelog

## v1.0.0 — Initial Release

### Added
- `UAdornComponent` — replicated wearable item manager with 10 body slots
- `UAdornItemData` — hard-reference Data Asset for wearable item definitions
- `UAdornSetData` — Data Asset defining set bonus thresholds
- Stat aggregation cache including set bonus contributions
- Set bonus activation and deactivation when piece count crosses thresholds
- Two visual modes: mesh attachment (socket-based) and mesh swap (component replacement)
- Visual mesh spawning and cleanup on equip/unequip
- Delegates: `OnItemEquipped`, `OnItemUnequipped`, `OnSetBonusActivated`, `OnSetBonusDeactivated`
- Full replication of slot contents and active set bonuses
- Weight tracking via `GetTotalWeight`
- Blueprint-callable API for all component functions
- No GAS dependency
