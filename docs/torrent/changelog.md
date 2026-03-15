# Torrent — Changelog

## v1.0.0 — Initial Release

### Added
- `UTorrentSubsystem` — `UWorldSubsystem` managing Niagara component pools per system asset
- Per-system active instance budgets via `SetPoolBudget`
- Global particle count budget via `SetGlobalParticleBudget`
- Four priority levels: Critical, High, Normal, Low
- Priority-based culling — lowest priority active effects are culled first when budget exceeded
- Pool recycling — deactivated components returned to free list, never destroyed
- `PreWarmPool` to pre-allocate pool entries and eliminate first-spawn hitches
- `SpawnLocal` for client-side cosmetic effects that don't replicate
- `UTorrentHandle` — lightweight object wrapping a pooled `UNiagaraComponent`
- `SetParameter` on handle for per-spawn Niagara parameter overrides
- Server-authoritative spawning with client replication
- Debug on-screen pool statistics via `r.Torrent.Debug 1`
- Blueprint-callable API for all subsystem functions
