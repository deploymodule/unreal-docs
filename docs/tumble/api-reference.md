# Tumble — API Reference

## UTumbleComponent

`#include "TumbleComponent.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `EnterRagdoll` | `void EnterRagdoll(FVector InitialImpulse, FName ImpulseBone)` | Transition to ragdoll state |
| `ExitRagdoll` | `void ExitRagdoll()` | Begin get-up sequence |
| `ApplyImpulse` | `void ApplyImpulse(FName Bone, FVector Impulse, EImpulseMode Mode)` | Apply bone impulse without full ragdoll |
| `IsRagdolling` | `bool IsRagdolling()` | True if in Ragdoll or Blending state |
| `GetState` | `ETumbleState GetState()` | Current state enum value |
| `GetRagdollPelvisLocation` | `FVector GetRagdollPelvisLocation()` | World-space pelvis location during ragdoll |
| `GetRagdollPelvisRotation` | `FRotator GetRagdollPelvisRotation()` | World-space pelvis rotation during ragdoll |
| `SetGetUpEnabled` | `void SetGetUpEnabled(bool bEnabled)` | Enable/disable automatic get-up |
| `SetGetUpDelay` | `void SetGetUpDelay(float Seconds)` | Seconds in ragdoll before auto get-up |
| `SetFaceUpMontage` | `void SetFaceUpMontage(UAnimMontage* Montage)` | Get-up animation (face up) |
| `SetFaceDownMontage` | `void SetFaceDownMontage(UAnimMontage* Montage)` | Get-up animation (face down) |
| `SetBlendProfile` | `void SetBlendProfile(UTumbleBlendProfile* Profile)` | Assign drive profile |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnEnterRagdoll` | `()` | Ragdoll state begins |
| `OnExitRagdoll` | `()` | Get-up sequence completes |
| `OnGetUpStarted` | `(bool bFaceUp)` | Get-up animation begins |

## ETumbleState

```cpp
UENUM(BlueprintType)
enum class ETumbleState : uint8
{
    Animated,
    Blending,
    Ragdoll,
    GetUp,
};
```

## EImpulseMode

```cpp
UENUM(BlueprintType)
enum class EImpulseMode : uint8
{
    Additive,     // Add to current velocity
    Override,     // Set bone velocity directly
};
```

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Tumble.ReplicationRate` | float | 10.0 | Bone transform broadcast rate (Hz) |
| `r.Tumble.StableVelocityThreshold` | float | 5.0 | Velocity below which ragdoll is considered stable |
| `r.Tumble.Debug` | int | 0 | Draw drive strength overlays |
