# Torrent — API Reference

## UTorrentSubsystem

`#include "TorrentSubsystem.h"`

### Spawn Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `Spawn` | `UTorrentHandle* Spawn(UNiagaraSystem* System, FVector Location, FRotator Rotation, ETorrentPriority Priority)` | Spawn at world location |
| `SpawnAttached` | `UTorrentHandle* SpawnAttached(UNiagaraSystem* System, USceneComponent* Parent, FName Socket, ETorrentPriority Priority)` | Spawn attached to component |
| `SpawnLocal` | `UTorrentHandle* SpawnLocal(UNiagaraSystem* System, FVector Location, FRotator Rotation)` | Local-only cosmetic spawn |
| `Release` | `void Release(UTorrentHandle* Handle)` | Return handle to pool |
| `ReleaseAll` | `void ReleaseAll(UNiagaraSystem* System)` | Release all instances of a system |

### Budget Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetPoolBudget` | `void SetPoolBudget(UNiagaraSystem* System, int32 MaxActive)` | Per-system active limit |
| `SetGlobalParticleBudget` | `void SetGlobalParticleBudget(int32 MaxParticles)` | Global particle cap |
| `PreWarmPool` | `void PreWarmPool(UNiagaraSystem* System, int32 Count)` | Pre-allocate pool entries |
| `GetActiveCount` | `int32 GetActiveCount(UNiagaraSystem* System)` | Current active count |
| `IsWithinBudget` | `bool IsWithinBudget(UNiagaraSystem* System)` | Check if budget allows spawn |
| `GetGlobalBudgetUsage` | `float GetGlobalBudgetUsage()` | 0–1 global usage fraction |

## UTorrentHandle

`#include "TorrentHandle.h"`

| Property/Function | Description |
|-------------------|-------------|
| `IsActive()` | True if effect is currently playing |
| `GetComponent()` | Underlying `UNiagaraComponent` |
| `SetParameter(Name, Value)` | Set Niagara parameter on the active component |
| `Priority` | ETorrentPriority assigned at spawn |

## ETorrentPriority

```cpp
UENUM(BlueprintType)
enum class ETorrentPriority : uint8
{
    Critical = 0,   // Never culled
    High     = 1,
    Normal   = 2,
    Low      = 3,   // Culled first
};
```

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Torrent.GlobalBudget` | int | 1000000 | Global particle count cap |
| `r.Torrent.Debug` | int | 0 | Draw active pool stats on screen |
