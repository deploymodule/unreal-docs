# FastFind — API Reference

## UFastFindSubsystem

`#include "FastFindSubsystem.h"`

### Registration

| Function | Signature | Description |
|----------|-----------|-------------|
| `RegisterActor` | `void RegisterActor(AActor* Actor)` | Add actor to grid |
| `UnregisterActor` | `void UnregisterActor(AActor* Actor)` | Remove actor from grid |
| `IsRegistered` | `bool IsRegistered(AActor* Actor)` | Check registration state |
| `GetRegisteredCount` | `int32 GetRegisteredCount()` | Total registered actors |

### Query Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `FindActorsInRadius` | `TArray<AActor*> FindActorsInRadius(FVector Origin, float Radius)` | All actors in sphere |
| `FindActorsInRadiusByTag` | `TArray<AActor*> FindActorsInRadiusByTag(FVector Origin, float Radius, FName Tag)` | Filter by actor tag |
| `FindNearestActor` | `AActor* FindNearestActor(FVector Origin, float Radius)` | Single nearest actor |
| `FindNearestActors` | `TArray<AActor*> FindNearestActors(FVector Origin, float Radius, int32 Count)` | N nearest actors sorted |
| `FindActorsInCone` | `TArray<AActor*> FindActorsInCone(FVector Origin, FVector Forward, float HalfAngleDeg, float Range)` | Cone-shaped query |

### Template Query

```cpp
template<typename T>
void FindActorsInRadiusByClass(FVector Origin, float Radius, TArray<T*>& OutActors);
```

## UFastFindComponent

`#include "FastFindComponent.h"`

Auto-registers owner actor with `UFastFindSubsystem` on `BeginPlay` and unregisters on `EndPlay`.

| Property | Type | Description |
|----------|------|-------------|
| `bAutoRegister` | bool | Auto-register on BeginPlay (default true) |
| `UpdatePriority` | EFastFindUpdatePriority | Static / Dynamic / Frequent |

## UFastFindBFL

`#include "FastFindBFL.h"` — Blueprint Function Library

All subsystem functions are available as static Blueprint nodes under the **FastFind** category.

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.FastFind.CellSize` | float | 500 | Grid cell size in cm |
| `r.FastFind.UpdateRate` | float | 30 | Position update rate in Hz |
| `r.FastFind.Debug` | int | 0 | Draw grid cell boundaries |
