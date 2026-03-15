# FastFind — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"FastFind"` to your `Build.cs`.

## 2. Register Actors

Actors must be registered with the grid to be queryable. Use `UFastFindComponent` for automatic registration:

```cpp
// Add to any actor you want to be findable
FastFindComponent = CreateDefaultSubobject<UFastFindComponent>(TEXT("FastFind"));
```

Or register manually:

```cpp
UFastFindSubsystem* FF = GetWorld()->GetSubsystem<UFastFindSubsystem>();
FF->RegisterActor(MyActor);
```

## 3. Find Actors in Radius

From Blueprint (BFL static call):

```
FastFind - Find Actors In Radius
  Origin: Player Location
  Radius: 800.0
  → OutActors: array of Actor
```

From C++:

```cpp
UFastFindSubsystem* FF = GetWorld()->GetSubsystem<UFastFindSubsystem>();
TArray<AActor*> NearbyActors = FF->FindActorsInRadius(Origin, 800.f);
```

## 4. Filter by Class

```cpp
TArray<AEnemy*> NearbyEnemies;
FF->FindActorsInRadiusByClass<AEnemy>(Origin, 800.f, NearbyEnemies);
```

## 5. Find Nearest Actor

```cpp
AActor* Nearest = FF->FindNearestActor(Origin, 1200.f);
if (Nearest) { /* target it */ }
```

## 6. Find N Nearest Actors

```cpp
TArray<AActor*> Nearest5 = FF->FindNearestActors(Origin, 2000.f, 5);
```

## 7. Cone Query

```cpp
// 60-degree cone, 1000cm range, forward from MyActor
TArray<AActor*> InCone = FF->FindActorsInCone(
    MyActor->GetActorLocation(),
    MyActor->GetActorForwardVector(),
    60.f,    // half-angle degrees
    1000.f   // range
);
```

## 8. Configure Grid

```ini
; Cell size in cm (larger cells = fewer cells to check but more false positives per cell)
r.FastFind.CellSize 500

; Grid update rate in Hz (lower = less CPU, slightly stale positions)
r.FastFind.UpdateRate 30
```
