# FastFind — Overview

## Architecture

```
UFastFindSubsystem  (UWorldSubsystem)
    └── FFastFindGrid   — flat hash grid (configurable cell size)
            ├── TMap<FIntVector, TArray<AActor*>>  CellMap
            └── TMap<AActor*, FIntVector>           ActorToCell
```

## Broadphase Grid

FastFind maintains a flat hash grid partitioned into cells. Each cell is a fixed-size cube in world space (default 500cm). Actors are assigned to a cell based on their current location. On each query, only the cells that overlap the query sphere are examined — typically 1–27 cells for a small radius.

### Why Not Physics Overlaps?

UE's `OverlapSphereForObjects` performs an accurate geometry test but requires the physics thread. For game logic queries (e.g., "find the 3 nearest enemies"), pixel-perfect accuracy is unnecessary and the physics sync overhead is wasteful. FastFind's grid is approximate (cell-boundary false positives exist) but a second exact distance check filters them out in constant time.

### Grid Update Strategy

Actors in the grid are updated lazily:
- On registration, the actor is placed in the correct cell
- On each FastFind subsystem tick (configurable rate), dirty actors (moved more than half a cell size since last update) are re-inserted
- Static actors are never re-inserted

## Query Types

| Query | Description |
|-------|-------------|
| `FindActorsInRadius` | All registered actors within a sphere |
| `FindActorsInRadiusByClass` | Same, filtered by class |
| `FindActorsInRadiusByTag` | Same, filtered by actor tag |
| `FindNearestActor` | Single nearest actor within radius |
| `FindNearestActors` | N nearest actors sorted by distance |
| `FindActorsInCone` | Actors within a forward-facing cone |

## Performance Characteristics

| Scenario | GetAllActors | OverlapSphere | FastFind |
|----------|-------------|---------------|----------|
| 1000 actors, radius 500cm | ~1ms | ~0.3ms | ~0.02ms |
| 1000 actors, radius 2000cm | ~1ms | ~1ms | ~0.08ms |
| Per-frame query on 100 units | ~100ms | ~30ms | ~2ms |

*Approximate figures — actual performance varies by CPU and scene.*
