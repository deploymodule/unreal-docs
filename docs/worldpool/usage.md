# WorldPool — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > WorldPool** — enable and restart.

## Step 2: Configure the Subsystem

In **Project Settings > WorldPool**:
- **Cell Size** — world units per grid cell (default: 5000).
- **Activation Radius** — cells within this distance of a player are active (default: 10000).
- **Delta Persistence Enabled** — toggle delta state serialization.

## Step 3: Add UWorldClassPoolComponent to Your Level Blueprint or Game Mode

=== "C++"
    ```cpp
    #include "WorldPool/WorldPoolSubsystem.h"

    UWorldPoolSubsystem* Pool = GetWorld()->GetSubsystem<UWorldPoolSubsystem>();

    // Register a pool for enemy actors
    Pool->RegisterClassPool(ANPCEnemy::StaticClass(), /* pool size */ 64);

    // Register a pool for debris props
    Pool->RegisterClassPool(ADebrisProp::StaticClass(), 128);
    ```

=== "Blueprint"
    Call **World Pool > Register Class Pool** in **Begin Play** with the actor class and pool size.

## Step 4: Add Spawn Requests to the Grid

```cpp
// Request a spawn at a world location — WorldPool manages actor lifecycle
FWorldPoolSpawnRequest Request;
Request.ActorClass = ANPCEnemy::StaticClass();
Request.Transform = FTransform(FRotator::ZeroRotator, FVector(3000, 1500, 0));
Request.InitialDeltaRecord = nullptr; // Optional — load from save data

Pool->AddSpawnRequest(Request);
```

In Blueprint: **World Pool > Add Spawn Request**.

## Step 5: Load Delta State from Save Data

On game load, restore delta records for previously-seen actors:

```cpp
TArray<FWorldPoolDeltaRecord> SavedDeltas = MySaveGame->LoadPoolDeltas();
for (auto& Delta : SavedDeltas)
{
    Pool->SetDeltaRecord(Delta.SpawnRequestId, Delta);
}
```

## Step 6: Manually Acquire and Release Actors

For scripted scenarios (cutscenes, boss arenas) you may need direct pool control:

```cpp
// Force-acquire an enemy actor
AActor* Enemy = Pool->AcquireActor(ANPCEnemy::StaticClass(), SpawnTransform);

// Return it when done
Pool->ReleaseActor(Enemy);
```

## Step 7: Subscribe to Pool Events

```cpp
Pool->OnActorAcquired.AddDynamic(this, &AMyGameMode::HandleActorSpawned);
Pool->OnActorReleased.AddDynamic(this, &AMyGameMode::HandleActorDespawned);
```

## Step 8: Save Delta Records

At game save time:

```cpp
TArray<FWorldPoolDeltaRecord> Deltas = Pool->GetAllDeltaRecords();
MySaveGame->SavePoolDeltas(Deltas);
```
