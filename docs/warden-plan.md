# Warden — AI State Tree Task Library

## Overview
Genre-agnostic, modular State Tree tasks, conditions, and EQS query templates for UE 5.7.
Ships both **AIPerception-coupled** adapters AND a **decoupled interface** so any custom perception system can plug in.

**Plugin Name:** Warden
**Location:** `Plugins/Warden/`
**Pattern:** Matches OmniSense's StateTree conventions (InstanceData structs, auto-resolution helpers, no TStateTreeExternalDataHandle)

---

## Architecture

### Dual Perception Strategy

```
┌──────────────────────────────────────────────┐
│           IWardenPerceptionProvider           │  ← UInterface
│  ─────────────────────────────────────────── │
│  FindTargets(SenseTag, Filter) → TArray<>    │
│  HasLineOfSight(Target, Channel) → bool      │
│  GetStimulusLocation(SenseTag) → FVector     │
│  GetStimulusStrength(Target) → float         │
│  GetPerceivedActors(SenseTag) → TArray<>     │
└──────────────┬───────────────┬───────────────┘
               │               │
    ┌──────────▼──┐    ┌───────▼──────────┐
    │ UWardenAI   │    │ UWardenCustom    │
    │ Perception  │    │ Perception       │
    │ Adapter     │    │ Adapter (stub)   │
    │ (wraps UAI  │    │ (users implement │
    │ Perception) │    │  or bind to own  │
    └─────────────┘    │  system)         │
                       └──────────────────┘
```

- `UWardenAIPerceptionAdapter` — ships ready-to-use, wraps `UAIPerceptionComponent`
- `UWardenCustomPerceptionAdapter` — abstract base; users subclass for OmniSense, custom systems, etc.
- Tasks bind to `IWardenPerceptionProvider` — they don't know which adapter is active
- If no explicit provider bound, auto-resolve: check owner pawn for `IWardenPerceptionProvider` implementor → fall back to `UAIPerceptionComponent` wrapper

### Noise Delegate Pattern
`ListenForNoise` exposes an optional `FWardenNoiseDelegate` input. If bound, it uses the delegate. If not, it listens to UE's built-in `AActor::MakeNoise` / `PawnNoiseEmitterComponent` system.

```cpp
DECLARE_DYNAMIC_DELEGATE_ThreeParams(FWardenNoiseDelegate,
    const FVector&, Location,
    AActor*, Instigator,
    float, Loudness);
```

### Auto-Resolution Helper
```cpp
// WardenSTHelpers.h — matches OmniSense pattern
IWardenPerceptionProvider* ResolvePerceptionProvider(
    FStateTreeExecutionContext& Context,
    TScriptInterface<IWardenPerceptionProvider> ExplicitProvider);
```

---

## Module Structure

```
Plugins/Warden/
├── Warden.uplugin
├── Source/
│   └── Warden/
│       ├── Warden.Build.cs
│       ├── Warden.h / .cpp                          (Module)
│       ├── WardenTypes.h                             (Enums, delegates, shared structs)
│       │
│       ├── Perception/
│       │   ├── WardenPerceptionProvider.h             (IWardenPerceptionProvider UInterface)
│       │   ├── WardenAIPerceptionAdapter.h/.cpp       (UAIPerceptionComponent wrapper)
│       │   └── WardenCustomPerceptionAdapter.h/.cpp   (Abstract base for custom systems)
│       │
│       ├── Tasks/
│       │   ├── WardenSTHelpers.h                      (Auto-resolution utilities)
│       │   ├── WardenTask_Movement.h/.cpp             (MoveTo, MoveToSpline, Patrol, Flee, RotateToFace)
│       │   ├── WardenTask_Timing.h/.cpp               (Wait, WaitRandom, Cooldown, Gate)
│       │   ├── WardenTask_Animation.h/.cpp            (PlayMontage, PlayMontageWaitNotify, SetMovementMode)
│       │   ├── WardenTask_Perception.h/.cpp           (FindTarget, HasLineOfSight, ListenForNoise)
│       │   ├── WardenTask_Spatial.h/.cpp              (RunEQSQuery, FindCover, FindRandomReachable)
│       │   ├── WardenTask_Interaction.h/.cpp          (SetFocus, SendGameplayEvent, SetBlackboardValue)
│       │   ├── WardenTask_Meta.h/.cpp                 (Log, SucceedAfter, FailAfter)
│       │   ├── WardenTask_Advanced.h/.cpp             (SmartObjectClaim/Use, Strafe, Leash, FormationSlot, TeleportTo)
│       │   ├── WardenTask_Squad.h/.cpp                (ShareTarget, BroadcastStimulus, SelectWeightedRandom)
│       │   ├── WardenTask_Pet.h/.cpp                  (FollowOwner, TeleportToOwner, MatchOwnerSpeed, IdleAmbient, FetchItem, StayCommand, HeelCommand, PetInteraction, Wander, AvoidCombatZone, EmotionalState, ReactToOwnerCombat)
│       │   └── WardenTask_Companion.h/.cpp            (CompanionFollow, CompanionEngage, CompanionRevive, CompanionHeal, CompanionCallout, CompanionContextAction, CompanionCombatPosition, CompanionAmmoShare, CompanionWaitAtDoor, CompanionMountVehicle, CompanionInvestigate, CompanionStayBack, AdjustAggressiveness, MatchPlayerPacing, CombatRoleSwitch)
│       │
│       ├── Conditions/
│       │   └── WardenConditions.h/.cpp                (IsTargetAlive, IsTargetInRange, HasGameplayTag, IsMoving, RandomChance, IsOnNavMesh, CompareFloat/Int/Enum, TimeSinceLastState)
│       │
│       └── Context/
│           ├── WardenPetContext.h                     (FWardenPetContext — owner, command, mood, timestamp)
│           ├── WardenCompanionContext.h                (FWardenCompanionContext — role, aggressiveness, command)
│           └── WardenSquadContext.h                    (FWardenSquadContext — shared target, formation, threat level)
│
└── Content/
    └── EQS/
        ├── EQS_Warden_FindCover.uasset
        ├── EQS_Warden_FindFlank.uasset
        ├── EQS_Warden_FindAmbush.uasset
        ├── EQS_Warden_FindGroupSpacing.uasset
        ├── EQS_Warden_FindHighGround.uasset
        ├── EQS_Warden_FindInteractable.uasset
        ├── EQS_Warden_FindRetreat.uasset
        ├── EQS_Warden_FindOverwatch.uasset
        └── EQS_Warden_FindNavLink.uasset
```

### Build.cs Dependencies
```csharp
PublicDependencyModuleNames.AddRange(new string[] {
    "Core", "CoreUObject", "Engine",
    "AIModule",
    "NavigationSystem",
    "StateTreeModule",
    "GameplayTags",
    "GameplayTasks",
});

// Optional — guarded by #if WITH_GAMEPLAY_ABILITIES
PrivateDependencyModuleNames.Add("GameplayAbilities");
```

### .uplugin Plugins Array
```json
"Plugins": [
    { "Name": "StateTree", "Enabled": true },
    { "Name": "GameplayStateTree", "Enabled": true }
]
```

---

## Phase Plan

### Phase 1 — Foundation + Core Tasks (25 tasks + 8 conditions)
1. Plugin scaffold (module, Build.cs, uplugin)
2. `WardenTypes.h` — enums, delegates, shared structs
3. `IWardenPerceptionProvider` interface
4. `UWardenAIPerceptionAdapter` — wraps UAIPerceptionComponent
5. `UWardenCustomPerceptionAdapter` — abstract stub
6. `WardenSTHelpers.h` — auto-resolution
7. **Movement tasks** — MoveTo, MoveToSpline, Patrol, Flee, RotateToFace
8. **Timing tasks** — Wait, WaitRandom, Cooldown, Gate
9. **Animation tasks** — PlayMontage, PlayMontageWaitNotify, SetMovementMode
10. **Perception tasks** — FindTarget, HasLineOfSight, ListenForNoise (dual adapter + noise delegate)
11. **Spatial tasks** — RunEQSQuery, FindCover, FindRandomReachable
12. **Interaction tasks** — SetFocus, SendGameplayEvent, SetBlackboardValue
13. **Meta tasks** — Log, SucceedAfter, FailAfter
14. **Conditions** — all 8
15. `IsInRange` condition (was listed as task, fits better as condition)

### Phase 2 — Advanced Modular (15 tasks + EQS templates)
16. SmartObjectClaim, SmartObjectUse
17. Strafe, Leash, FormationSlot
18. TeleportTo, WaitForGameplayTag
19. BroadcastStimulusToGroup, SelectWeightedRandom, TimeLimitDecorator
20. ShareTargetWithSquad, PredictTargetPosition
21. PlayVOLine, AdjustThreatLevel, WaitForPreviousTaskNotify
22. `WardenSquadContext.h`
23. EQS query data assets (9 templates — created in editor, shipped as Content/)

### Phase 3 — Pet Behavior (12 tasks)
24. `WardenPetContext.h` — FWardenPetContext struct
25. FollowOwner, TeleportToOwner, MatchOwnerSpeed
26. IdleAmbient, ReactToOwnerCombat, EmotionalState
27. FetchItem, StayCommand, HeelCommand
28. PetInteraction, Wander, AvoidCombatZone

### Phase 4 — Companion / Sidekick Behavior (15 tasks)
29. `WardenCompanionContext.h` — FWardenCompanionContext struct
30. CompanionFollow, CompanionEngage, CompanionRevive
31. CompanionHeal, CompanionCallout, CompanionContextAction
32. CompanionCombatPosition, CompanionAmmoShare, CompanionWaitAtDoor
33. CompanionMountVehicle, CompanionInvestigate, CompanionStayBack
34. AdjustAggressiveness, MatchPlayerPacing, CombatRoleSwitch

---

## Task Manifest (85 total)

### Core Movement (5)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_MoveTo` | WardenTask_Movement | Navigate to actor/location, acceptance radius, strafe option |
| `FWardenTask_MoveToSpline` | WardenTask_Movement | Follow spline component at configurable speed |
| `FWardenTask_Patrol` | WardenTask_Movement | Cycle waypoints (loop, ping-pong, random) |
| `FWardenTask_Flee` | WardenTask_Movement | Move away from threat with min distance |
| `FWardenTask_RotateToFace` | WardenTask_Movement | Smooth rotation to face target |

### Timing & Flow (4)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_Wait` | WardenTask_Timing | Fixed duration wait |
| `FWardenTask_WaitRandom` | WardenTask_Timing | Random duration in range |
| `FWardenTask_Cooldown` | WardenTask_Timing | Prevent re-entry for N seconds |
| `FWardenTask_Gate` | WardenTask_Timing | Bool gate, optional latch |

### Animation (3)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_PlayMontage` | WardenTask_Animation | Play and wait for completion/interrupt |
| `FWardenTask_PlayMontageWaitNotify` | WardenTask_Animation | Play, succeed on named notify |
| `FWardenTask_SetMovementMode` | WardenTask_Animation | Switch movement mode |

### Perception (3)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_FindTarget` | WardenTask_Perception | Query via IWardenPerceptionProvider OR AIPerception |
| `FWardenTask_HasLineOfSight` | WardenTask_Perception | Trace-based LOS, configurable channel/bone |
| `FWardenTask_ListenForNoise` | WardenTask_Perception | MakeNoise listener OR custom delegate |

### Spatial (3)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_RunEQSQuery` | WardenTask_Spatial | General EQS query execution |
| `FWardenTask_FindCover` | WardenTask_Spatial | Threat-relative cover query |
| `FWardenTask_FindRandomReachable` | WardenTask_Spatial | NavMesh random point in radius |

### Interaction (3)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_SetFocus` | WardenTask_Interaction | Set/clear AI focus |
| `FWardenTask_SendGameplayEvent` | WardenTask_Interaction | Broadcast gameplay event tag |
| `FWardenTask_SetBlackboardValue` | WardenTask_Interaction | Write to blackboard (BT bridge) |

### Meta (3)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_Log` | WardenTask_Meta | Debug log with binding substitution |
| `FWardenTask_SucceedAfter` | WardenTask_Meta | Force success after N seconds |
| `FWardenTask_FailAfter` | WardenTask_Meta | Force fail after N seconds |

### Advanced Modular (15)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_SmartObjectClaim` | WardenTask_Advanced | Find and claim SmartObject slot |
| `FWardenTask_SmartObjectUse` | WardenTask_Advanced | Execute claimed slot behavior |
| `FWardenTask_Strafe` | WardenTask_Advanced | Circle target maintaining distance |
| `FWardenTask_Leash` | WardenTask_Advanced | Succeed while within leash radius |
| `FWardenTask_FormationSlot` | WardenTask_Advanced | Claim formation position |
| `FWardenTask_TeleportTo` | WardenTask_Advanced | Instant reposition with effects |
| `FWardenTask_WaitForGameplayTag` | WardenTask_Squad | Block until tag added/removed |
| `FWardenTask_BroadcastStimulus` | WardenTask_Squad | Push stimulus to group |
| `FWardenTask_SelectWeightedRandom` | WardenTask_Squad | Weighted random selection |
| `FWardenTask_TimeLimitDecorator` | WardenTask_Squad | Force transition after duration |
| `FWardenTask_ShareTarget` | WardenTask_Squad | Write target to squad context |
| `FWardenTask_PredictTargetPosition` | WardenTask_Squad | Velocity-based position prediction |
| `FWardenTask_PlayVOLine` | WardenTask_Squad | Voice-over with cooldown/priority |
| `FWardenTask_AdjustThreatLevel` | WardenTask_Squad | Modify threat float on context |
| `FWardenTask_WaitForTaskNotify` | WardenTask_Squad | Listen for delegate from prior state |

### Pet Behavior (12)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_FollowOwner` | WardenTask_Pet | Follow at offset, match speed |
| `FWardenTask_TeleportToOwner` | WardenTask_Pet | Snap when leash exceeded |
| `FWardenTask_MatchOwnerSpeed` | WardenTask_Pet | Blend locomotion to match owner |
| `FWardenTask_IdleAmbient` | WardenTask_Pet | Random idle montages on timer |
| `FWardenTask_ReactToOwnerCombat` | WardenTask_Pet | Transition on owner combat events |
| `FWardenTask_FetchItem` | WardenTask_Pet | Pick up → deliver to owner |
| `FWardenTask_StayCommand` | WardenTask_Pet | Lock position until released |
| `FWardenTask_HeelCommand` | WardenTask_Pet | Tight follow, combat-ready |
| `FWardenTask_PetInteraction` | WardenTask_Pet | Paired anim on owner input |
| `FWardenTask_Wander` | WardenTask_Pet | Random points near owner |
| `FWardenTask_AvoidCombatZone` | WardenTask_Pet | Flee threats, stay in leash |
| `FWardenTask_EmotionalState` | WardenTask_Pet | Mood system drives behavior |

### Companion / Sidekick (15)
| Task | File | Description |
|------|------|-------------|
| `FWardenTask_CompanionFollow` | WardenTask_Companion | Tactical follow, avoid aim line |
| `FWardenTask_CompanionEngage` | WardenTask_Companion | Attack target with priority rules |
| `FWardenTask_CompanionRevive` | WardenTask_Companion | Navigate to player, revive montage |
| `FWardenTask_CompanionHeal` | WardenTask_Companion | Heal player/self below threshold |
| `FWardenTask_CompanionCallout` | WardenTask_Companion | VO bark with cooldown |
| `FWardenTask_CompanionContextAction` | WardenTask_Companion | World interaction (doors, terminals) |
| `FWardenTask_CompanionCombatPosition` | WardenTask_Companion | Role-based combat positioning |
| `FWardenTask_CompanionAmmoShare` | WardenTask_Companion | Share resources with player |
| `FWardenTask_CompanionWaitAtDoor` | WardenTask_Companion | Stack up at chokepoints |
| `FWardenTask_CompanionMountVehicle` | WardenTask_Companion | Find seat, enter vehicle |
| `FWardenTask_CompanionInvestigate` | WardenTask_Companion | Move to marked location, report |
| `FWardenTask_CompanionStayBack` | WardenTask_Companion | Hold position, provide overwatch |
| `FWardenTask_AdjustAggressiveness` | WardenTask_Companion | Dynamic passive↔aggressive float |
| `FWardenTask_MatchPlayerPacing` | WardenTask_Companion | Exploration follow, avoid blocking |
| `FWardenTask_CombatRoleSwitch` | WardenTask_Companion | Swap tank/dps/support sets |

### Conditions (9)
| Condition | Description |
|-----------|-------------|
| `FWardenCond_IsTargetAlive` | Health > 0, not pending kill |
| `FWardenCond_IsTargetInRange` | Distance threshold check |
| `FWardenCond_HasGameplayTag` | Tag container query |
| `FWardenCond_IsMoving` | Velocity magnitude threshold |
| `FWardenCond_RandomChance` | Float 0-1 probability gate |
| `FWardenCond_IsOnNavMesh` | Validate actor on navmesh |
| `FWardenCond_CompareValue` | Generic float/int/enum comparison |
| `FWardenCond_TimeSinceLastState` | Elapsed since named state exit |
| `FWardenCond_IsInRange` | Distance between two actors/locations |

### EQS Templates (9 data assets)
| Asset | Scoring |
|-------|---------|
| `EQS_Warden_FindCover` | Trace from threat, dot product filter |
| `EQS_Warden_FindFlank` | 90°+ angle from threat forward |
| `EQS_Warden_FindAmbush` | Proximity to predicted path + LOS + concealment |
| `EQS_Warden_FindGroupSpacing` | Min distance from squad members |
| `EQS_Warden_FindHighGround` | Z advantage + reachability |
| `EQS_Warden_FindInteractable` | SmartObject/interface tag filter |
| `EQS_Warden_FindRetreat` | Distance from threat + proximity to safe zone |
| `EQS_Warden_FindOverwatch` | LOS coverage area + elevation |
| `EQS_Warden_FindNavLink` | Closest nav link (traversal) |

---

## Totals

| Category | Tasks | Conditions | EQS | Total |
|----------|-------|------------|-----|-------|
| Core | 24 | 9 | 3 | 36 |
| Advanced | 15 | — | 7 | 22 |
| Pet | 12 | — | — | 12 |
| Companion | 15 | — | — | 15 |
| **Total** | **66** | **9** | **10** | **85** |
