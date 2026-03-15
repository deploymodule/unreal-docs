# Flux — API Reference

## UFluxSubsystem

`#include "FluxSubsystem.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `RegisterActor` | `void RegisterActor(AActor* Actor)` | Track actor against realm volumes |
| `UnregisterActor` | `void UnregisterActor(AActor* Actor)` | Stop tracking actor |
| `GetActorInfluence` | `float GetActorInfluence(AActor* Actor, FName RealmID)` | 0–1 influence of realm on actor |
| `GetActorDominantRealm` | `FName GetActorDominantRealm(AActor* Actor)` | Highest-influence realm |
| `GetAllActorInfluences` | `TMap<FName,float> GetAllActorInfluences(AActor* Actor)` | All realm influences |
| `RegisterRealm` | `void RegisterRealm(FName RealmID)` | Declare a realm (optional, auto-created on volume spawn) |
| `GetAllRealms` | `TArray<FName> GetAllRealms()` | All registered realm IDs |

## UFluxComponent

`#include "FluxComponent.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `GetInfluence` | `float GetInfluence(FName RealmID)` | Current realm influence on this actor |
| `GetDominantRealm` | `FName GetDominantRealm()` | Current dominant realm |
| `GetAllInfluences` | `TMap<FName,float> GetAllInfluences()` | All realm influences |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnDominantRealmChanged` | `(FName OldRealm, FName NewRealm)` | Dominant realm changes |
| `OnInfluenceChanged` | `(FName RealmID, float OldValue, float NewValue)` | Influence changes above threshold |
| `OnVolumeEntered` | `(AFluxVolume* Volume)` | Actor enters a volume |
| `OnVolumeExited` | `(AFluxVolume* Volume)` | Actor exits a volume |

## AFluxVolume

| Property | Type | Description |
|----------|------|-------------|
| `RealmID` | FName | Which realm this volume belongs to |
| `ShapeType` | EFluxShape | Box / Sphere / Capsule |
| `Influence` | float | Influence value at volume center (0–1) |
| `FalloffRadius` | float | World-space blend edge width |

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Flux.QueryInterval` | float | 0.1 | Influence update interval (seconds) |
| `r.Flux.ChangeThreshold` | float | 0.01 | Min delta to fire OnInfluenceChanged |
| `r.Flux.Debug` | int | 0 | Draw volume bounds and influence values |
