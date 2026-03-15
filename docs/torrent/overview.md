# Torrent — Overview

## Architecture

```
UTorrentSubsystem  (UWorldSubsystem)
    ├── TMap<UNiagaraSystem*, FTorrentPool>  Pools
    │       ├── TArray<UTorrentHandle*>  FreeList
    │       └── TArray<UTorrentHandle*>  ActiveList
    └── FTorrentBudget  GlobalBudget

UTorrentHandle  (UObject, pooled)
    ├── UNiagaraComponent*  Component
    ├── ETorrentPriority    Priority
    └── FTorrentSpawnParams Params
```

## Pooling

Each `UNiagaraSystem` asset gets its own pool. When you request a spawn:

1. Torrent checks the free list for a dormant component
2. If found: resets parameters, activates, returns handle
3. If not found and budget allows: spawns new component, adds to pool
4. If not found and budget exceeded: culls the lowest-priority active effect, recycles it

When an effect finishes (or you call `Release`), the component is deactivated and returned to the free list. Niagara components are never destroyed — only activated/deactivated.

## Budget System

Torrent enforces two levels of budget:

### Per-System Budget

Each Niagara system can have a max active instance count set in its `FTorrentPoolConfig`:

```cpp
UTorrentSubsystem::SetPoolBudget(ExplosionSystem, 8);  // max 8 concurrent explosions
```

### Global Particle Budget

An optional global particle count budget (approximate, based on max particle counts declared in each system's metadata). When the global budget is exceeded, the lowest-priority effects are culled system-wide.

## Priority Culling

Effects are assigned a priority at spawn time:

| Priority | Description | Culling behavior |
|----------|-------------|-----------------|
| `Critical` | Player VFX, important hits | Never culled |
| `High` | Enemy hits, projectiles | Culled last |
| `Normal` | Ambient effects, footsteps | Culled before High |
| `Low` | Background ambience, distant fires | Culled first |

## Replication

Torrent manages effect spawn on the server. Spawn RPCs replicate to clients, where the client-side Torrent subsystem handles the visual. Clients can also spawn local-only effects (cosmetic non-gameplay VFX) that don't replicate.
