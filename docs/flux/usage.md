# Flux — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Flux"` to your `Build.cs`.

## 2. Place Flux Volumes

1. Drag an `AFluxVolume` into the level
2. In the Details panel, set:
   - **Realm ID**: `"Fire"`
   - **Shape**: Sphere (radius 800)
   - **Influence**: 1.0
   - **Falloff Radius**: 200

Place multiple volumes to build out the realm's territory.

## 3. Add Flux Component to Actors

```cpp
FluxComponent = CreateDefaultSubobject<UFluxComponent>(TEXT("Flux"));
```

The component auto-registers with `UFluxSubsystem` on `BeginPlay`.

## 4. Query Influence

```cpp
UFluxComponent* Flux = GetComponentByClass<UFluxComponent>();

float FireInfluence = Flux->GetInfluence(FName("Fire"));
FName Dominant     = Flux->GetDominantRealm();
```

## 5. Bind to Events

```cpp
FluxComponent->OnDominantRealmChanged.AddDynamic(this, &AMyChar::OnRealmChanged);

void AMyChar::OnRealmChanged(FName OldRealm, FName NewRealm)
{
    if (NewRealm == "Fire")
    {
        StartBurningEffect();
    }
}
```

## 6. Direct Subsystem Query (Without Component)

```cpp
UFluxSubsystem* Flux = GetWorld()->GetSubsystem<UFluxSubsystem>();
float IcePct = Flux->GetActorInfluence(MyActor, FName("Ice"));
```

## 7. Register and Unregister Manually

```cpp
// Manual registration (if not using UFluxComponent)
Flux->RegisterActor(MyActor);
Flux->UnregisterActor(MyActor);
```

## 8. Configure Query Rate

```ini
; Update influence values every 0.1 seconds (10 Hz)
r.Flux.QueryInterval 0.1
```

Reduce the query interval for performance-critical scenes with many registered actors.
