# WorldPool — Changelog

## v1.0.0 — Initial Release

### Added
- `UWorldPoolSubsystem` with spatial grid, activation radius, class pool registry, and spawn request management.
- `UWorldClassPoolComponent` with O(1) free-list acquire/release for each actor class.
- `FWorldPoolGrid` spatial hash grid with configurable cell size and per-player activation radius.
- `FWorldPoolSpawnRequest` — logical spawn records decoupled from actual actor instances.
- Delta persistence via `FWorldPoolDeltaRecord` — compact serialized per-actor state for save/restore.
- `SetDeltaRecord` / `GetAllDeltaRecords` APIs for save game integration.
- Manual pool control via `AcquireActor` / `ReleaseActor` for scripted scenarios.
- Replication integration: pooled actors removed from replication graph when released; re-added on acquire.
- `OnActorAcquired`, `OnActorReleased`, `OnCellActivated`, `OnCellDeactivated` delegates.
- `GetPoolStats` for runtime monitoring and performance profiling.
- Full Blueprint exposure for all subsystem and pool component functions.
- `r.WorldPool.Debug` CVar for grid visualization and pool utilization overlay.
