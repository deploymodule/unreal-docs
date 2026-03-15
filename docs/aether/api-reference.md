# Aether — API Reference

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Aether.Enabled` | int | 0 | Master enable switch |
| `r.Aether.BlendStrength` | float | 1.0 | AKF / original blend (0=original, 1=full AKF) |
| `r.Aether.KernelRadius` | float | 5.0 | Filter kernel radius in pixels |
| `r.Aether.Sharpness` | float | 4.0 | Sector weight exponent (higher = sharper edges) |
| `r.Aether.OrientationSensitivity` | float | 1.0 | Kernel elongation along structure tensor |
| `r.Aether.SectorCount` | int | 8 | Number of filter sectors (4 or 8) |
| `r.Aether.TensorSmoothRadius` | float | 2.0 | Structure tensor Gaussian blur radius |
| `r.Aether.ResScale` | float | 1.0 | AKF pass resolution scale (0.5 for half-res) |

## UAetherSubsystem

`#include "AetherSubsystem.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetEnabled` | `void SetEnabled(bool bEnabled)` | Master on/off |
| `SetBlendStrength` | `void SetBlendStrength(float Strength)` | AKF / original blend factor |
| `SetKernelRadius` | `void SetKernelRadius(float Pixels)` | Stroke kernel size |
| `SetSharpness` | `void SetSharpness(float Alpha)` | Edge sharpness exponent |
| `SetOrientationSensitivity` | `void SetOrientationSensitivity(float Sensitivity)` | Kernel elongation strength |
| `SetSectorCount` | `void SetSectorCount(int32 Count)` | 4 or 8 sectors |
| `SetTensorSmoothRadius` | `void SetTensorSmoothRadius(float Pixels)` | Tensor blur radius |
| `GetSettings` | `UAetherSettings* GetSettings()` | Mutable settings object |

## UAetherSettings

| Property | Type | Description |
|----------|------|-------------|
| `bEnabled` | bool | Master enable |
| `BlendStrength` | float | 0–1 blend with original |
| `KernelRadius` | float | Kernel radius pixels |
| `Sharpness` | float | Weighting exponent |
| `OrientationSensitivity` | float | Anisotropy elongation |
| `SectorCount` | int32 | 4 or 8 sectors |
| `TensorSmoothRadius` | float | Structure tensor smooth radius |
| `ResScale` | float | Render resolution scale |

## AAetherPostProcessVolume

Actor that applies Aether settings within a bounded volume.

| Property | Type | Description |
|----------|------|-------------|
| `Settings` | FAetherVolumeSettings | Per-volume override settings |
| `BlendRadius` | float | World-space blend falloff at volume edge |
| `Priority` | int32 | Volume priority when overlapping |
