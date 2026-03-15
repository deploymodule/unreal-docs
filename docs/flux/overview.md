# Flux — Overview

## Architecture

```
UFluxSubsystem  (UWorldSubsystem)
    ├── TMap<FName, FFluxRealm>         Realms
    │       └── TArray<AFluxVolume*>    Volumes per realm
    └── TMap<AActor*, FFluxActorState>  RegisteredActors

AFluxVolume  (AActor)
    ├── FName RealmID
    ├── UBoxComponent / USphereComponent / UCapsuleComponent
    ├── float Influence          — 0–1 weight at volume center
    └── float FalloffRadius      — world-space blend edge

UFluxComponent  (UActorComponent)
    ├── TMap<FName, float>   RealmInfluences  — current influence per realm
    └── FName                DominantRealm    — highest-influence realm
```

## Realms

A realm is a named layer of the world. You define as many realms as your game needs:

- `"Fire"`, `"Ice"`, `"Void"` — elemental biomes
- `"FactionA"`, `"FactionB"` — territory control
- `"StealthDetection"` — enemy detection radii
- `"NightWorld"`, `"DayWorld"` — parallel planes

Realms are registered at runtime; no upfront enumeration is needed.

## Zone Volumes

`AFluxVolume` actors define the spatial extent of a realm. Each volume has:
- A **RealmID** it belongs to
- A **shape** (box, sphere, or capsule)
- An **influence** value (0–1) at the volume's center
- A **falloff radius** for smooth blending at the volume edge

Multiple overlapping volumes in the same realm are summed (capped at 1.0).

## Actor Registration

Actors with a `UFluxComponent` are automatically registered when `BeginPlay` fires. The subsystem tracks their location against all active realm volumes on a configurable tick interval (`r.Flux.QueryInterval`).

## Influence Query

The subsystem computes per-realm influence for each registered actor:
- 0.0 = actor is fully outside all volumes of that realm
- 1.0 = actor is fully inside a volume at peak influence

`DominantRealm` is the realm with the highest current influence value.

## Delegates

| Delegate | Fires when |
|----------|------------|
| `OnDominantRealmChanged` | Actor's dominant realm changes |
| `OnInfluenceChanged` | Any realm's influence changes by more than threshold |
| `OnVolumeEntered` | Actor enters a specific volume |
| `OnVolumeExited` | Actor exits a specific volume |
