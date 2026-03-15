# Warden — API Reference

## FWardenTask_Base

Base struct all Warden tasks derive from.

| Virtual | Description |
|---|---|
| `EnterState` | Called when the owning state is entered. Return `Running` to continue ticking. |
| `TickTask` | Called every frame while the state is active. Return `Succeeded` or `Failed` to exit. |
| `ExitState` | Called when the state exits for any reason. |

## Selected Task Reference

### Movement Tasks

| Task | Key Properties | Description |
|---|---|---|
| `FWardenTask_MoveToTarget` | `TargetActor`, `AcceptanceRadius` | Navigates to the target actor using NavMesh. |
| `FWardenTask_Patrol` | `PatrolPath`, `bRandomOrder` | Loops through a patrol path asset. |
| `FWardenTask_Strafe` | `StrafeRadius`, `StrafeSpeed` | Circles the target while maintaining facing. |
| `FWardenTask_FleeFromThreat` | `FleeDistance`, `ThreatActor` | Navigates away from a threat actor. |
| `FWardenTask_TakeCover` | `CoverQuery`, `MaxSearchRadius` | Runs an EQS cover query and moves to the result. |

### Combat Tasks

| Task | Key Properties | Description |
|---|---|---|
| `FWardenTask_AttackMelee` | `MontageToPlay`, `HitRadius` | Plays a melee montage and applies damage on hit window. |
| `FWardenTask_AttackRanged` | `ProjectileClass`, `FireRate` | Spawns projectiles at the configurable fire rate. |
| `FWardenTask_ReloadWeapon` | `ReloadMontage`, `ReloadTime` | Plays reload montage and sets a Blackboard ammo key. |
| `FWardenTask_BreakEngagement` | `DisengageDistance` | Navigates away from combat to a safe distance. |

### Perception Tasks

| Task | Key Properties | Description |
|---|---|---|
| `FWardenTask_WaitForSight` | `TargetActor`, `Timeout` | Waits until the target is in line-of-sight or times out. |
| `FWardenTask_RememberLastKnown` | `TargetActor` | Writes the target's last known position to the memory layer. |
| `FWardenTask_BroadcastAlert` | `AlertRadius`, `AlertTag` | Notifies nearby AI agents via Relay or a Gameplay Message. |

## Conditions

| Condition | Key Properties | Pass Condition |
|---|---|---|
| `FWardenCond_IsTargetAlive` | `TargetActor` | Target has health > 0 or no health component. |
| `FWardenCond_IsTargetInRange` | `TargetActor`, `Range` | Distance to target is ≤ `Range`. |
| `FWardenCond_IsTargetVisible` | `TargetActor` | Line trace to target is unobstructed. |
| `FWardenCond_IsHealthBelow` | `Threshold` | AI health component reports value below threshold. |
| `FWardenCond_HasMemory` | `MemoryKey` | Memory layer has a non-expired entry for the given key. |
| `FWardenCond_RandomChance` | `Probability` | Random float < `Probability` each evaluation. |

## UWardenMemoryComponent

| Property / Function | Type | Description |
|---|---|---|
| `MemoryDecayRate` | `float` | Confidence units lost per second on all memories. |
| `MaxMemoryAge` | `float` | Memories older than this (in seconds) are purged. |
| `RememberActor` | `void (AActor*, FName Key, float Confidence)` | Stores or updates a memory entry for an actor. |
| `ForgetActor` | `void (AActor*)` | Immediately removes all memories about an actor. |
| `GetMemoryConfidence` | `float (AActor*)` | Returns the current confidence for the most recent memory of an actor. |
| `GetLastKnownLocation` | `FVector (AActor*)` | Returns the last known world location from memory. |
