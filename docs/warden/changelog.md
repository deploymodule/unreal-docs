# Warden — Changelog

## v1.0.0 — Initial Release

### Added
- 66 StateTree task implementations across Movement (14), Perception (9), Combat (18), Pet/Companion (12), and Utility (13) categories.
- 9 reusable `FWardenCond_*` condition structs.
- Five EQS query template assets: `FindCoverPoint`, `FindFlankPosition`, `FindPatrolPoint`, `FindPickupNearby`, `FindOpenGround`.
- `UWardenMemoryComponent` with configurable decay rate, max age, and per-actor confidence API.
- Dual perception architecture — sense layer feeds memory layer; all tasks read from memory.
- `FWardenTask_Base` base struct for custom task authoring with `EnterState`/`TickTask`/`ExitState` virtuals.
- Full Blueprint property exposure on all tasks and conditions.
- `WardenStateTreeSchema` for AI StateTree assets (optional — all tasks work with the default schema too).
