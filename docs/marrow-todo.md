# Marrow Plugin — Implementation Roadmap

SRS-driven todo list. Each module maps to a class and file.
Check off as implemented. Stubs exist for all declared functions.

---

## Infrastructure
- [x] `Marrow.uplugin`
- [x] `Source/Marrow/Marrow.Build.cs`
- [x] `Source/Marrow/Public/Marrow.h` + `Private/Marrow.cpp` (module lifecycle)

---

## Module I — Visibility & Spatial Math (`UMarrowVisibilityLibrary`)
**File:** `Public/MarrowVisibilityLibrary.h` · `Private/MarrowVisibilityLibrary.cpp`

- [x] **1. Frustum Visibility Check** — `MarrowFrustumVisibilityCheck`
  - `ULocalPlayer::GetProjectionData` → `ComputeViewProjectionMatrix()`
  - `GetViewFrustumBounds()` → `FConvexVolume`
  - `AActor::GetActorBounds()` → `FConvexVolume::IntersectBox()`
  - COMPLETE. Near-plane toggle param included.

- [x] **2. Screen Space Proximity** — `MarrowScreenSpaceProximity`
  - `APlayerController::ProjectWorldLocationToScreen()`
  - `GetViewportSize()` → screen centre = `(W/2, H/2)`
  - `FVector2D::Distance()` → pixel distance
  - Returns `-1.0` when point is behind camera

- [x] **3. Illumination Sensor** — `MarrowIlluminationSensor`
  - `TActorIterator<ALight>` to find nearby lights (Point, Spot, Directional)
  - `LineTraceSingleByChannel` per light for occlusion check
  - Inverse-square law: `Contribution = Intensity / (Dist * Dist)`
  - Normalise against reference brightness; `FMath::Clamp` to [0,1]
  - Dependencies: `Engine` (already in Build.cs)

- [x] **4. Blindspot Spawner** — `MarrowBlindspotSpawner`
  - `UNavigationSystemV1::GetRandomReachablePointInRadius()` for NavMesh candidates
  - Reuse Module 1 frustum construction but test raw `FVector` via `IntersectSphere(Pt, 0.f)`
  - Loop up to `MaxAttempts` — return first off-screen NavMesh point
  - Dependencies: `NavigationSystem` (already in Build.cs)

- [x] **5. Constriction Fit Validator** — `MarrowConstrictionFitValidator`
  - `FCollisionShape::MakeCapsule(Radius, HalfHeight)`
  - `UWorld::SweepMultiByChannel()` along Entry→Exit vector
  - Blocking hit check on results array
  - Build `OutEntryTransform` / `OutExitTransform` from sweep direction quaternion

---

## Module II — Audio & Procedural Kinematics

### 6. Acoustic Path Occlusion (`UMarrowAudioLibrary`)
**Files to create:** `Public/MarrowAudioLibrary.h` · `Private/MarrowAudioLibrary.cpp`

- [x] `MarrowAcousticPathOcclusion`
  - `UNavigationSystemV1::FindPathToLocationSynchronously()` — gets nav path object
  - Walk `FNavPathPoint` array, sum segment lengths → total path length
  - `Attenuation = 1.0 - FMath::Clamp(PathLength / MaxRadius, 0, 1)`
  - Returns normalised float — plug into audio component volume multiplier

### 7. Procedural Kinematics Component (`UMarrowProceduralKinematicsComponent`)
**Files to create:** `Public/MarrowProceduralKinematicsComponent.h` · `Private/MarrowProceduralKinematicsComponent.cpp`

- [x] Enum `EMarrowKinematicState` — `Continuous`, `Burst`, `KinematicBlend`
- [x] `TickComponent` drives all modes
- [x] **Continuous:** `FMath::PerlinNoise1D(Time * FrequencyX)` per axis, multiply by AmplitudeVector, add to relative transform
- [x] **Burst:** Same noise but gated by a duration timer; resets to zero after BurstDuration seconds
- [x] **KinematicBlend:** Cache parent animated transform each frame. On `TriggerKinematicHit()`:
  - Detach visual mesh, apply noise/impulse
  - Spring-damper (semi-implicit Euler, k=SpringStiffness, b=SpringDamping) drives offset back to zero
  - Blend completes when delta is below threshold; re-attach mesh
- [x] Exposed properties: `FrequencyVector`, `AmplitudeVector`, `BurstDuration`, `BlendSpeed`, `SpringStiffness`, `SpringDamping`

---

## Module III — AI & Logic (`UMarrowAILibrary`)
**Files to create:** `Public/MarrowAILibrary.h` · `Private/MarrowAILibrary.cpp`

- [x] **8. Tension Hysteresis Evaluator** — `MarrowTensionHysteresisEvaluator`
  - Signature: `(float CurrentTension, float TargetTension, float DeltaTime, float RiseSpeed, float DecaySpeed) → float`
  - `float Speed = (TargetTension > CurrentTension) ? RiseSpeed : DecaySpeed;`
  - `return FMath::FInterpTo(CurrentTension, TargetTension, DeltaTime, Speed);`
  - Caller stores and feeds back `CurrentTension` each tick (same pattern as FastMath Spring)
  - `EMarrowInteractionResult` enum declared; Module 9 & 10 stubs compile

- [x] **9. Predator Intercept Vector** — `MarrowPredatorInterceptVector`
  - `FVector ProjectedPos = TargetVelocity * LookaheadTime + TargetLocation`
  - `UNavigationSystemV1::ProjectPointToNavigation(ProjectedPos, NavLoc, nullptr, &QueryExtent)`
  - Returns `NavLoc.Location` on success; returns `ProjectedPos` (not `TargetLocation`) if no nav point found
  - Dependencies: `NavigationSystem` (already in Build.cs)

- [x] **10. Probability Interaction Router** — `MarrowProbabilityInteractionRouter`
  - Enum `EMarrowInteractionResult` — `Success`, `Fail`, `CriticalEvent`
  - UFUNCTION with `ExpandEnumAsExecs = "OutResult"` meta tag; `OutResult` is first param (C++ default-value constraint)
  - `float Roll = FMath::FRand();` — uniform [0,1)
  - `ScaledCritical` clamped to `1 - ScaledSuccess` so thresholds never sum beyond 1
  - Compare roll against `ScaledSuccess`, `ScaledSuccess + ScaledCritical`

---

## Module IV — Advanced Components & Frameworks

### 11. Fixation Gaze Component (`UMarrowFixationGazeComponent`)
**Files to create:** `Public/MarrowFixationGazeComponent.h` · `Private/MarrowFixationGazeComponent.cpp`

- [x] `TickComponent` — per-frame gaze test
- [x] Get camera position + forward via `APlayerController` → `PlayerCameraManager`
- [x] `FVector ToObject = (OwnerLocation - CameraPos).GetSafeNormal()`
- [x] `float Dot = FVector::DotProduct(ToObject, CameraForward)`
- [x] If `Dot >= FMath::Cos(FMath::DegreesToRadians(ToleranceAngleDegrees))` → increment `GazeTimer`
- [x] Else → reset `GazeTimer`
- [x] When `GazeTimer >= RequiredGazeSeconds` → `OnGazeThresholdReached.Broadcast()` (multicast delegate)
- [x] `bFireOnce` property to prevent repeated broadcasts
- [x] Properties: `ToleranceAngleDegrees` (default 15°), `RequiredGazeSeconds`, `bFireOnce`
- [x] `ResetGaze()` to re-arm bFireOnce; `GetGazeProgress()` returns normalised [0,1] for VFX/UI

### 12. Unseen Asset Swapper (`UMarrowUnseenAssetSwapperComponent`)
**Files to create:** `Public/MarrowUnseenAssetSwapperComponent.h` · `Private/MarrowUnseenAssetSwapperComponent.cpp`

- [x] `TArray<TObjectPtr<AActor>> PrimaryActors` — actors that must ALL be off-screen to trigger
- [x] `TArray<TObjectPtr<AActor>> SecondaryActors` — actors to enable when primary goes off-screen
- [x] `TickComponent` — polls `UMarrowVisibilityLibrary::MarrowFrustumVisibilityCheck()` (Module 1) per PrimaryActor
- [x] When ALL primary actors return `false` (none visible):
  - `for each Primary: SetActorHiddenInGame(true); SetActorEnableCollision(false)`
  - `for each Secondary: SetActorHiddenInGame(false); SetActorEnableCollision(true)`
  - `bSwapped = true` — tick returns immediately until Reset()
- [x] `Reset()` function to restore state (re-show primaries, re-hide secondaries, resume polling)
- [x] `bHasValidPrimary` guard — all-null PrimaryActors array does not trigger the swap

### 13. Async Action Sequencer (`UMarrowAsyncActionSequencer`)
**Files to create:** `Public/MarrowAsyncActionSequencer.h` · `Private/MarrowAsyncActionSequencer.cpp`

- [x] Inherit `UBlueprintAsyncActionBase`
- [x] `USTRUCT FMarrowSequencedAction` — fields: `EMarrowActionType ActionType` (enum), `TSoftObjectPtr<UObject> Asset`, `float Delay`
- [x] `EMarrowActionType` enum: `PlayLevelSequence`, `PlaySound`, `Wait`, `CustomEvent`
- [x] `UFUNCTION(BlueprintCallable, meta=(BlueprintInternalUseOnly="true"))` factory `static UMarrowAsyncActionSequencer* RunSequence(...)`
- [x] `TArray<FMarrowSequencedAction> ActionQueue` + `int32 CurrentIndex`
- [x] `Activate()` → calls `StepNext()`
- [x] `StepNext()`:
  - If `CurrentIndex >= ActionQueue.Num()` → `OnCompleted.Broadcast()`, `SetReadyToDestroy()`
  - Load asset via `TSoftObjectPtr::LoadSynchronous()` (or async)
  - For `PlayLevelSequence`: create `ULevelSequencePlayer`, `Player->OnFinished.AddDynamic(this, &ThisClass::OnStepFinished)`
  - For `Wait`: `GetWorld()->GetTimerManager().SetTimer(...)` → `OnStepFinished`
  - For `PlaySound`: `UAudioComponent::OnAudioFinished` delegate → `OnStepFinished`
- [x] `OnStepFinished()` → `++CurrentIndex; StepNext()`
- [x] Output pins: `OnStepStarted`, `OnCompleted` (UPROPERTY multicast delegates)
- [x] Dependencies: `LevelSequence`, `MovieScene` (already in Build.cs)

---

## Notes

### Build.cs Dependencies (all already added)
| Need              | Module             |
|-------------------|--------------------|
| NavMesh queries   | `NavigationSystem` |
| AI perception     | `AIModule`         |
| Level Sequences   | `LevelSequence`, `MovieScene`, `MovieSceneTracks` |

### Shared Utility Needed (not a module, internal helper)
- Consider a `MarrowFrustumUtils` internal helper that extracts the frustum
  construction steps (Modules 1, 4, 12 all build the same frustum).
  Would live in `Private/` and be `#included` only in .cpp files.

### Testing Checklist (per module)
- [ ] Compiles with zero warnings
- [ ] Blueprint node appears in correct category
- [ ] Thumbnail/tooltip readable in BP editor
- [ ] Tested in PIE with a basic actor
- [ ] Documented edge cases (null inputs, no NavMesh, zero extents)
