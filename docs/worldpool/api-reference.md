# WorldPool — API Reference

## UWorldPoolSubsystem

| Function | Signature | Description |
|---|---|---|
| `RegisterClassPool` | `void (TSubclassOf<AActor>, int32 PoolSize)` | Creates and preallocates a pool for the given actor class. |
| `UnregisterClassPool` | `void (TSubclassOf<AActor>)` | Destroys all pooled instances for the given class. |
| `AddSpawnRequest` | `FGuid (FWorldPoolSpawnRequest)` | Adds a logical spawn request to the grid. Returns a request GUID for tracking. |
| `RemoveSpawnRequest` | `void (FGuid RequestId)` | Removes a spawn request; returns any active actor to the pool. |
| `AcquireActor` | `AActor* (TSubclassOf<AActor>, FTransform)` | Manually acquires an actor from the pool and places it at the given transform. |
| `ReleaseActor` | `void (AActor*)` | Returns an actor to the pool, saving its delta state. |
| `SetDeltaRecord` | `void (FGuid RequestId, FWorldPoolDeltaRecord)` | Pre-loads a delta record for a spawn request (for save game restoration). |
| `GetDeltaRecord` | `FWorldPoolDeltaRecord (FGuid RequestId)` | Returns the current delta record for a spawn request. |
| `GetAllDeltaRecords` | `TArray<FWorldPoolDeltaRecord> ()` | Returns all delta records — useful for save game serialization. |
| `GetPoolStats` | `FWorldPoolStats (TSubclassOf<AActor>)` | Returns pool statistics: total size, in-use count, free count. |

### Delegates

| Delegate | Parameters | Description |
|---|---|---|
| `OnActorAcquired` | `AActor*, FGuid RequestId` | Fired when an actor is acquired from the pool. |
| `OnActorReleased` | `AActor*, FGuid RequestId` | Fired when an actor is returned to the pool. |
| `OnCellActivated` | `FIntPoint CellCoord` | Fired when a grid cell enters the activation radius. |
| `OnCellDeactivated` | `FIntPoint CellCoord` | Fired when a grid cell exits the activation radius. |

## UWorldClassPoolComponent

Manages the free-list for a single actor class. Attached to an internal manager actor by `UWorldPoolSubsystem`.

| Function | Signature | Description |
|---|---|---|
| `Acquire` | `AActor* (FTransform)` | Pops an actor from the free list and places it at the transform. |
| `Release` | `void (AActor*)` | Returns an actor to the free list. |
| `GetFreeCount` | `int32 ()` | Number of actors currently in the free list. |
| `GetInUseCount` | `int32 ()` | Number of actors currently acquired. |

## FWorldPoolGrid

Spatial grid structure. Accessed via `UWorldPoolSubsystem`.

| Function | Signature | Description |
|---|---|---|
| `GetCellForLocation` | `FIntPoint (FVector)` | Returns the grid cell coordinates for a world location. |
| `GetActiveCells` | `TArray<FIntPoint> ()` | Returns all currently active cell coordinates. |
| `IscellActive` | `bool (FIntPoint)` | Returns true if the cell is within the activation radius of any player. |

## FWorldPoolSpawnRequest (Struct)

| Field | Type | Description |
|---|---|---|
| `ActorClass` | `TSubclassOf<AActor>` | The actor class to spawn from the pool. |
| `Transform` | `FTransform` | World transform for placement on activation. |
| `InitialDeltaRecord` | `FWorldPoolDeltaRecord*` | Optional delta to apply on first acquisition (for save restoration). |

## FWorldPoolDeltaRecord (Struct)

| Field | Type | Description |
|---|---|---|
| `SpawnRequestId` | `FGuid` | Which spawn request this delta belongs to. |
| `DeltaData` | `TArray<uint8>` | Compact serialized delta state (custom per actor class). |
| `LastSaveTimestamp` | `double` | UTC time when this delta was last written. |

## FWorldPoolStats (Struct)

| Field | Type | Description |
|---|---|---|
| `TotalPoolSize` | `int32` | Total preallocated actor count. |
| `InUseCount` | `int32` | Currently acquired actors. |
| `FreeCount` | `int32` | Available actors in the free list. |
