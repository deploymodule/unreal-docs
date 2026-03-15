# Gimbal — API Reference

## UGimbalComponent

The core runtime component. Attach to any `AController` subclass.

| Function | Signature | Description |
|---|---|---|
| `PushMode` | `void PushMode(UGimbalModeAsset* Mode, EGimbalBlend Blend)` | Pushes a camera mode onto the priority stack and begins blending. |
| `PopMode` | `void PopMode(UGimbalModeAsset* Mode)` | Removes the first stack entry matching the given asset. |
| `ClearStack` | `void ClearStack()` | Removes all modes and resets to no active camera. |
| `ApplyPreset` | `void ApplyPreset(UGimbalPresetAsset* Preset)` | Replaces the stack with the modes defined in a preset asset. |
| `BeginLockOn` | `void BeginLockOn(AActor* InitialTarget)` | Starts lock-on targeting. If `InitialTarget` is null the nearest valid target is chosen. |
| `EndLockOn` | `void EndLockOn()` | Ends lock-on and resumes normal camera behavior. |
| `CycleLockOnTarget` | `void CycleLockOnTarget(EGimbalCycleDir Dir)` | Cycles to the next or previous valid lock-on candidate. |
| `AdjustOrbitZoom` | `void AdjustOrbitZoom(float Delta)` | Changes orbit zoom level by `Delta` units, clamped to mode asset limits. |
| `GetActiveMode` | `UGimbalModeAsset* GetActiveMode() const` | Returns the highest-priority currently active mode. |
| `IsLockedOn` | `bool IsLockedOn() const` | Returns true if lock-on is currently active. |
| `GetLockOnTarget` | `AActor* GetLockOnTarget() const` | Returns the current lock-on target actor, or null. |

## UGimbalBehavior (Base Class)

All behaviors derive from this. Override `TickBehavior` to produce a `FGimbalOutput`.

| Virtual | Description |
|---|---|
| `TickBehavior` | Called every frame while the owning mode is active. Receives `FGimbalContext` and writes `FGimbalOutput`. |
| `OnModeActivated` | Called when the parent mode is pushed onto the stack. |
| `OnModeDeactivated` | Called when the parent mode is popped or the stack is cleared. |

## UGimbalModeAsset

| Property | Type | Description |
|---|---|---|
| `Priority` | `int32` | Higher values take precedence when blending with other active modes. |
| `BehaviorStack` | `TArray<TSubclassOf<UGimbalBehavior>>` | Ordered list of behaviors to instantiate for this mode. |
| `BlendInDuration` | `float` | Seconds to blend in when pushed. |
| `BlendOutDuration` | `float` | Seconds to blend out when popped. |
| `BlendCurve` | `UCurveFloat*` | Optional custom ease curve for blending. |
| `bAdditiveBlend` | `bool` | If true this mode composes additively on top of lower-priority modes. |

## UGimbalPresetAsset

| Property | Type | Description |
|---|---|---|
| `PresetName` | `FName` | Human-readable identifier. |
| `Modes` | `TArray<UGimbalModeAsset*>` | Ordered list of modes that make up this preset. |

## EGimbalBlend

| Value | Description |
|---|---|
| `Linear` | Linear interpolation over `BlendInDuration`. |
| `Curve` | Uses `BlendCurve` from the mode asset; falls back to linear if null. |
| `Instant` | No interpolation — snaps immediately. |

## EGimbalCycleDir

| Value | Description |
|---|---|
| `Next` | Cycles to the next candidate in sorted order. |
| `Previous` | Cycles to the previous candidate. |

## FGimbalContext (Struct)

Passed to every `TickBehavior` call. Read-only.

| Field | Type | Description |
|---|---|---|
| `ControlledPawn` | `APawn*` | The pawn currently possessed by the owning controller. |
| `LockOnTarget` | `AActor*` | Current lock-on target, or null. |
| `DeltaTime` | `float` | Frame delta time in seconds. |
| `ViewRotation` | `FRotator` | Current controller view rotation. |
