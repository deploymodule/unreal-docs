# Warden — Overview

## Design Goals

Warden is a plug-and-play StateTree task library. It ships with ready-to-use tasks for the most common AI behaviors so you spend time designing AI personalities rather than writing boilerplate patrol, chase, and flee logic.

## Dual Perception Architecture

Warden separates AI knowledge into two distinct layers:

```mermaid
graph LR
    SenseLayer[Sense Layer\nWhat AI currently detects] --> MemoryLayer[Memory Layer\nWhat AI remembers]
    MemoryLayer --> StateTree[StateTree Evaluation]
    StateTree --> Tasks[Warden Tasks]
```

- **Sense Layer** — real-time input from AIPerception or OmniSense. Raw, unfiltered stimulus events.
- **Memory Layer** — persistent knowledge store. Events age, confidence decays, and memories expire. Tasks read from memory, not the sense layer directly.

## Task Categories

| Category | Count | Examples |
|---|---|---|
| Movement | 14 | MoveToTarget, Patrol, Strafe, FleeFromThreat, Takecover |
| Perception | 9 | WaitForSight, RememberLastKnownPosition, BroadcastAlertToSquad |
| Combat | 18 | AttackMelee, AttackRanged, ReloadWeapon, BreakEngagement, CoverFire |
| Pet / Companion | 12 | FollowOwner, WaitForOwner, PlayIdleAnimation, FetchItem |
| Utility | 13 | WaitDelay, PlaySound, SetBlackboardKey, DebugLog, RandomBranch |

## Conditions (9)

| Condition | Description |
|---|---|
| `FWardenCond_IsTargetAlive` | Passes if the target actor is not dead (checks health component if present). |
| `FWardenCond_IsTargetInRange` | Passes if the target is within a configurable range. |
| `FWardenCond_IsTargetVisible` | Passes if the AI has line-of-sight to the target. |
| `FWardenCond_IsHealthBelow` | Passes if the AI's health is below a threshold. |
| `FWardenCond_HasMemory` | Passes if the memory layer has a valid entry for a given key. |
| `FWardenCond_IsAtLocation` | Passes if the AI is within tolerance of a world location. |
| `FWardenCond_IsPlayerNear` | Passes if any player pawn is within detection range. |
| `FWardenCond_IsItemAvailable` | Passes if a query-defined item actor exists in the world. |
| `FWardenCond_RandomChance` | Passes with a configurable probability each evaluation. |

## EQS Templates

Warden ships five pre-built EQS query templates: `FindCoverPoint`, `FindFlankPosition`, `FindPatrolPoint`, `FindPickupNearby`, and `FindOpenGround`. Drop them into any StateTree that uses EQS-based tasks.
