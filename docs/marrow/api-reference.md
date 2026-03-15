# Marrow — API Reference

## UMarrowSanityComponent

| Function | Signature | Description |
|----------|-----------|-------------|
| `DrainSanity` | `void DrainSanity(float Amount)` | Instant sanity loss |
| `RestoreSanity` | `void RestoreSanity(float Amount)` | Instant sanity gain |
| `SetDrainActive` | `void SetDrainActive(bool bActive, float Rate)` | Continuous drain toggle |
| `GetSanity` | `float GetSanity()` | Current sanity value (0–100) |
| `GetSanityNormalized` | `float GetSanityNormalized()` | 0.0–1.0 |
| `SetRecoveryRate` | `void SetRecoveryRate(float PerSecond)` | Safe-area recovery speed |

### Delegates

| Delegate | Signature |
|----------|-----------|
| `OnSanityThreshold` | `(float CurrentSanity, float Threshold)` |
| `OnSanityBroken` | `()` — fired when sanity reaches zero |

## UMarrowDreadComponent

| Function | Signature | Description |
|----------|-----------|-------------|
| `SpikeDread` | `void SpikeDread(float Amount)` | Add to dread immediately |
| `GetDread` | `float GetDread()` | Current dread (0–100) |
| `GetDreadNormalized` | `float GetDreadNormalized()` | 0.0–1.0 |
| `SetDecayRate` | `void SetDecayRate(float PerSecond)` | How fast dread dissipates |

## UMarrowPerceptionComponent

| Property/Function | Type | Description |
|-------------------|------|-------------|
| `HearingRange` | float | Hearing radius (cm) |
| `VisionConeAngle` | float | Full cone angle (degrees) |
| `VisionRange` | float | Max vision range (cm) |
| `LightSensitivity` | float | Multiplier on vision range based on scene luma |
| `OnPlayerDetected` | Delegate | Fired when player first detected |
| `OnPlayerLost` | Delegate | Fired when player leaves all sense ranges |

## UMarrowEnvironmentComponent

| Function | Signature | Description |
|----------|-----------|-------------|
| `GetHorrorPressure` | `float GetHorrorPressure()` | Composite horror 0–1 |
| `GetDarknessLevel` | `float GetDarknessLevel()` | Scene luma darkness 0–1 |
| `GetIsolationTime` | `float GetIsolationTime()` | Seconds without friendly contact |

## UMarrowFlickerSubsystem

| Function | Signature | Description |
|----------|-----------|-------------|
| `RegisterLight` | `void RegisterLight(ULightComponent* Light, UMarrowFlickerProfile* Profile)` | Add light to flicker system |
| `UnregisterLight` | `void UnregisterLight(ULightComponent* Light)` | Remove light |
| `SetFlickerEnabled` | `void SetFlickerEnabled(ULightComponent* Light, bool bEnabled)` | Toggle flicker |
| `SetGlobalIntensity` | `void SetGlobalIntensity(float Intensity)` | Scale all flicker effects |
