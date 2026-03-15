# Optica — API Reference

## UOpticaComponent

`#include "OpticaComponent.h"`

### Sway

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetSwayProfile` | `void SetSwayProfile(UOpticaSwayProfile* Profile)` | Assign sway parameters |
| `SetSwayEnabled` | `void SetSwayEnabled(bool bEnabled)` | Toggle sway |
| `SetSwayScale` | `void SetSwayScale(float Scale)` | Global sway multiplier |

### Recoil

| Function | Signature | Description |
|----------|-----------|-------------|
| `ApplyRecoil` | `void ApplyRecoil(UOpticaRecoilProfile* Profile)` | Apply one shot of recoil |
| `ResetRecoil` | `void ResetRecoil()` | Snap springs back to rest |
| `SetRecoilScale` | `void SetRecoilScale(float Scale)` | Global recoil multiplier |

### ADS

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetADS` | `void SetADS(bool bAiming)` | Enter/exit aim down sights |
| `IsADS` | `bool IsADS()` | Current ADS state |
| `SetADSBlendSpeed` | `void SetADSBlendSpeed(float Speed)` | ADS lerp speed |
| `SetADSFOV` | `void SetADSFOV(float FOV)` | Target FOV when aiming |

### Breathing

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetBreathingEnabled` | `void SetBreathingEnabled(bool bEnabled)` | Toggle breathing layer |
| `SetBreathingRate` | `void SetBreathingRate(float Hz)` | Breath cycles per second |
| `SetBreathingAmplitude` | `void SetBreathingAmplitude(float CM)` | Breath offset amplitude |
| `SetExertionLevel` | `void SetExertionLevel(float Level)` | 0–1 exertion affecting breath rate |
| `HoldBreath` | `void HoldBreath(float Duration)` | Suppress breathing for ADS hold |

### Footstep Bob

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetFootstepBobEnabled` | `void SetFootstepBobEnabled(bool bEnabled)` | Toggle bob |
| `SetFootstepBobFrequency` | `void SetFootstepBobFrequency(float Hz)` | Bob frequency |
| `SetFootstepBobAmplitude` | `void SetFootstepBobAmplitude(float CM)` | Bob height |

## CVars (Sensor Emulation)

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Optica.Sensor.Enabled` | int | 0 | Enable sensor emulation pass |
| `r.Optica.Sensor.NoiseIntensity` | float | 0.02 | Sensor grain intensity |
| `r.Optica.Sensor.ExposureLag` | float | 0.3 | Exposure adaptation speed (seconds) |
| `r.Optica.Sensor.MotionBlurScale` | float | 1.5 | Motion blur multiplier at high velocity |
| `r.Optica.Sensor.VignettePulse` | float | 0.0 | Heartbeat vignette strength |
