# Torrent — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Torrent"` to your `Build.cs`.

## 2. Spawn an Effect

Replace `UNiagaraFunctionLibrary::SpawnSystemAtLocation` with:

```cpp
UTorrentSubsystem* Torrent = GetWorld()->GetSubsystem<UTorrentSubsystem>();

UTorrentHandle* Handle = Torrent->Spawn(
    ExplosionSystem,            // UNiagaraSystem*
    HitLocation,                // FVector Location
    FRotator::ZeroRotator,      // FRotator Rotation
    ETorrentPriority::High      // Priority
);
```

## 3. Spawn Attached to an Actor

```cpp
UTorrentHandle* Handle = Torrent->SpawnAttached(
    FireSystem,
    MyActor->GetRootComponent(),
    NAME_None,             // socket
    ETorrentPriority::Normal
);
```

## 4. Release an Effect Manually

```cpp
// Before the effect naturally completes (e.g., fire stops when extinguished)
Torrent->Release(Handle);
```

## 5. Set a Pool Budget

```cpp
// At game startup or level load
Torrent->SetPoolBudget(BloodSplatterSystem, 16);  // max 16 simultaneous
Torrent->SetPoolBudget(ExplosionSystem, 4);        // max 4 simultaneous
```

## 6. Set Global Budget

```cpp
// Limit total approximate particle count
Torrent->SetGlobalParticleBudget(500000);
```

## 7. Pre-Warm Pools

Avoid first-spawn hitches by pre-creating pool entries at level start:

```cpp
Torrent->PreWarmPool(BloodSplatterSystem, 16);
Torrent->PreWarmPool(ExplosionSystem, 4);
```

## 8. Query Budget State

```cpp
int32 ActiveExplosions = Torrent->GetActiveCount(ExplosionSystem);
bool bBudgetOK         = Torrent->IsWithinBudget(ExplosionSystem);
float GlobalUsage      = Torrent->GetGlobalBudgetUsage();  // 0–1
```

## 9. Local-Only Spawn (Client Cosmetic)

```cpp
UTorrentHandle* LocalHandle = Torrent->SpawnLocal(FootstepSystem, FootLocation, FRotator::ZeroRotator);
```

Local spawns are never replicated and are culled more aggressively.
