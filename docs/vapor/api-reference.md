# Vapor — API Reference

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Vapor.Enabled` | int | 0 | Master enable switch (1 = on) |
| `r.Vapor.Density` | float | 0.3 | Global fog density |
| `r.Vapor.HeightFalloff` | float | 0.5 | Exponential height falloff coefficient |
| `r.Vapor.ScatterColorR/G/B` | float | 1/0.97/0.93 | Per-channel scatter tint |
| `r.Vapor.PhaseG` | float | 0.6 | Henyey-Greenstein g term |
| `r.Vapor.SkyGlowIntensity` | float | 1.0 | Sky dome in-scatter multiplier |
| `r.Vapor.ResolutionScale` | float | 0.5 | Scatter pass resolution (0.25–1.0) |
| `r.Vapor.RaymarchSteps` | int | 32 | Ray-march step count per pixel |
| `r.Vapor.MaxAdditionalLights` | int | 8 | Cap on registered scatter lights |

## UVaporSubsystem

`#include "VaporSubsystem.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetEnabled` | `void SetEnabled(bool bEnabled)` | Toggles fog scattering on/off |
| `SetDensity` | `void SetDensity(float Density)` | Sets global fog density |
| `SetHeightFalloff` | `void SetHeightFalloff(float Falloff)` | Sets height falloff coefficient |
| `SetScatterColor` | `void SetScatterColor(FLinearColor Color)` | Sets scatter tint |
| `SetPhaseG` | `void SetPhaseG(float G)` | Sets Henyey-Greenstein anisotropy |
| `SetSkyGlowIntensity` | `void SetSkyGlowIntensity(float Intensity)` | Sets sky dome contribution |
| `RegisterScatterLight` | `void RegisterScatterLight(ULightComponent* Light)` | Adds a light to the scatter list |
| `UnregisterScatterLight` | `void UnregisterScatterLight(ULightComponent* Light)` | Removes a light from the scatter list |
| `GetSettings` | `UVaporSettings* GetSettings()` | Returns the mutable settings object |

## UVaporSettings

`UObject` subclass. Exposes all parameters as `UPROPERTY` for Blueprint/editor access.

| Property | Type | Description |
|----------|------|-------------|
| `bEnabled` | bool | Master enable |
| `Density` | float | Fog density (0–1) |
| `HeightFalloff` | float | Height falloff |
| `ScatterColor` | FLinearColor | Scatter tint |
| `PhaseG` | float | Anisotropy g term |
| `SkyGlowIntensity` | float | Sky dome multiplier |
| `ResolutionScale` | float | Half-res scale |
| `RaymarchSteps` | int32 | Step count |

## AVaporFogVolume

Actor that defines a local fog density region.

| Property | Type | Description |
|----------|------|-------------|
| `DensityOverride` | float | Local density override value |
| `BlendRadius` | float | World-space blend falloff at volume edge |
| `bAdditiveBlend` | bool | Add to global density instead of overriding |
