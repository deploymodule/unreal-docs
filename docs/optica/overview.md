# Optica — Overview

## System Layers

```
UOpticaComponent  (UActorComponent)
    ├── FWeaponSway         — lag, tilt, and offset driven by input velocity
    ├── FRecoilSystem       — spring-based camera and weapon kick
    ├── FADSTransition      — smooth ADS/hipfire blend with FOV adjustment
    ├── FBreathingLayer     — procedural breathing cycle
    ├── FFootstepBob        — camera bob synced to character movement
    └── FSensorEmulation    — visual sensor noise, exposure, and vignette

UOpticaViewExtension  (FSceneViewExtensionBase)
    └── SensorEmulation render pass  — noise, exposure lag, motion blur scale
```

## Weapon Sway

Weapon sway is computed from the delta of the player's mouse/look input. Three channels contribute:

- **Lag sway** — weapon follows the camera with a delay (spring-lag offset)
- **Tilt sway** — horizontal input tilts the weapon (lateral roll)
- **Vertical sway** — vertical input offsets weapon position slightly

Each channel has configurable stiffness, damping, and max clamp. The combined sway is applied as a relative offset to the weapon socket transform.

## Recoil System

Recoil is modeled as a pair of critically-damped springs:

1. **Camera spring** — kicks the camera view up and sideways on fire
2. **Weapon spring** — kicks the weapon mesh back and upward independently

Recoil profiles are defined in `UOpticaRecoilProfile` data assets. Per-shot recoil is added to the spring target; the spring oscillates back to rest naturally.

## ADS Transition

When aiming down sights:
- Weapon socket interpolates to ADS position/rotation
- FOV lerps to the ADS FOV
- Sway and bob magnitudes scale down (more stable when aimed)
- Breathing effect reduces slightly

## Sensor Emulation

A render-thread post-process pass adds:
- **Sensor noise** — film-grain style noise scaled by low-light conditions
- **Exposure lag** — slow scene adaptation to brightness changes
- **Motion blur scale** — bodycam-style increased motion blur at high angular velocity
- **Vignette pulse** — heartbeat-synced vignette for stress feedback

## Breathing

Procedural breathing adds a subtle sine-wave sway to the weapon and camera. Breathing rate and amplitude increase with exertion (configurable via `SetExertionLevel`). In ADS, breathing amplitude is reduced for tactical realism.
