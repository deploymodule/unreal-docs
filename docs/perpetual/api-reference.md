# Perpetual — API Reference

## UPerpetualAnimationComponent

| Function | Signature | Description |
|---|---|---|
| `GoToStage` | `void GoToStage(FName StageName)` | Begins transitioning to the named stage immediately. |
| `Advance` | `void Advance()` | Moves to the next stage as defined in the current stage's `NextStage` field. |
| `Reverse` | `void Reverse()` | Transitions to the current stage's `ReverseStage`. |
| `SetAnimationAsset` | `void SetAnimationAsset(UPerpetualAnimationAsset* Asset)` | Replaces the active animation asset at runtime. |
| `SetTargetComponent` | `void SetTargetComponent(USceneComponent* Target)` | Changes the scene component being animated. |
| `GetCurrentStageName` | `FName GetCurrentStageName() const` | Returns the name of the currently active stage. |
| `IsTransitioning` | `bool IsTransitioning() const` | Returns true if a stage transition is in progress. |
| `GetStageProgress` | `float GetStageProgress() const` | Returns normalized progress (0.0–1.0) of the current transition. |
| `Pause` | `void Pause()` | Pauses the current transition in place. |
| `Resume` | `void Resume()` | Resumes a paused transition. |

### Delegates

| Delegate | Signature | Description |
|---|---|---|
| `OnStageBegin` | `FName StageName` | Broadcast at the start of a stage transition. |
| `OnStageComplete` | `FName StageName` | Broadcast when a stage reaches its target transform. |
| `OnSequenceComplete` | *(none)* | Broadcast when the final non-looping stage completes. |

## UPerpetualAnimationAsset

| Property | Type | Description |
|---|---|---|
| `Stages` | `TArray<FPerpetualStage>` | Ordered list of animation stages. |
| `DefaultStage` | `FName` | Stage to enter when the component first activates. |

## FPerpetualStage (Struct)

| Field | Type | Description |
|---|---|---|
| `StageName` | `FName` | Unique name within the asset. |
| `TargetTransform` | `FTransform` | Relative transform the component animates toward. |
| `Duration` | `float` | Transition duration in seconds. |
| `EaseCurve` | `UCurveFloat*` | Optional easing curve; linear if null. |
| `NextStage` | `FName` | Stage to auto-transition to on completion (empty = stop). |
| `NextStageDelay` | `float` | Seconds to wait after completion before auto-transitioning. |
| `ReverseStage` | `FName` | Stage to transition to when `Reverse()` is called. |
| `bLoop` | `bool` | If true, this stage restarts from the beginning on completion. |
| `BeginAudioCue` | `USoundBase*` | Sound to play when this stage begins. |
| `CompleteAudioCue` | `USoundBase*` | Sound to play when this stage completes. |

## FPerpetualStageState (Replicated Struct)

| Field | Type | Description |
|---|---|---|
| `CurrentStageName` | `FName` | Stage name on the server. |
| `ElapsedTime` | `float` | Elapsed time within the stage at the moment of replication. |
| `bPaused` | `bool` | Whether the transition is paused. |
