# Lucent — API Reference

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Lucent.Enabled` | int | 0 | Master enable |
| `r.Lucent.DOF.Enabled` | int | 0 | Depth of field enable |
| `r.Lucent.DOF.FocalLength` | float | 50 | Lens focal length (mm) |
| `r.Lucent.DOF.Aperture` | float | 2.8 | F-stop value |
| `r.Lucent.DOF.FocusDistance` | float | 300 | Focus distance (cm) |
| `r.Lucent.DOF.SensorWidth` | float | 36 | Film/sensor width (mm) |
| `r.Lucent.DOF.ResScale` | float | 0.5 | DOF pass resolution scale |
| `r.Lucent.Bloom.Enabled` | int | 0 | Physical bloom enable |
| `r.Lucent.Bloom.Intensity` | float | 0.5 | Bloom strength |
| `r.Lucent.Bloom.Threshold` | float | 1.0 | HDR threshold for bloom |
| `r.Lucent.Halation.Enabled` | int | 0 | Halation enable |
| `r.Lucent.Halation.Intensity` | float | 0.2 | Halation strength |
| `r.Lucent.Streaks.Enabled` | int | 0 | Anamorphic streaks enable |
| `r.Lucent.Streaks.Length` | float | 80 | Streak length in pixels |
| `r.Lucent.Streaks.Count` | int | 2 | Number of streaks (1/2/4) |
| `r.Lucent.CA.Enabled` | int | 0 | Chromatic aberration enable |
| `r.Lucent.CA.Strength` | float | 0.003 | CA displacement amount |
| `r.Lucent.Distortion.Enabled` | int | 0 | Lens distortion enable |
| `r.Lucent.Distortion.K1` | float | 0 | Brown-Conrady k1 coefficient |
| `r.Lucent.Distortion.K2` | float | 0 | Brown-Conrady k2 coefficient |

## ULucentSubsystem

`#include "LucentSubsystem.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetEnabled` | `void SetEnabled(bool bEnabled)` | Master on/off |
| `SetFocalLength` | `void SetFocalLength(float MM)` | Lens focal length |
| `SetAperture` | `void SetAperture(float FStop)` | F-stop aperture |
| `SetFocusDistance` | `void SetFocusDistance(float CM)` | Focus plane distance |
| `SetSensorWidth` | `void SetSensorWidth(float MM)` | Sensor/film width |
| `SetBokehShape` | `void SetBokehShape(ULucentBokehAsset* Asset)` | Custom bokeh texture |
| `SetAutoFocusTarget` | `void SetAutoFocusTarget(AActor* Target)` | Auto-focus actor |
| `SetAutoFocusSpeed` | `void SetAutoFocusSpeed(float Speed)` | Auto-focus lerp speed |
| `SetStreaksEnabled` | `void SetStreaksEnabled(bool bEnabled)` | Toggle streak flares |
| `SetStreakLength` | `void SetStreakLength(float Pixels)` | Streak pixel length |
| `SetStreakCount` | `void SetStreakCount(ELucentStreakCount Count)` | 1/2/4 streaks |
| `SetStreakColor` | `void SetStreakColor(FLinearColor Color)` | Streak tint |
| `GetSettings` | `ULucentSettings* GetSettings()` | Mutable settings object |

## ULucentSettings

| Property | Type | Description |
|----------|------|-------------|
| `bEnabled` | bool | Master enable |
| `FocalLength` | float | Focal length mm |
| `Aperture` | float | F-stop |
| `FocusDistance` | float | Focus cm |
| `SensorWidth` | float | Sensor mm |
| `BokehAsset` | ULucentBokehAsset* | Custom bokeh shape |
| `bStreaksEnabled` | bool | Streak enable |
| `StreakLength` | float | Streak pixels |
| `StreakColor` | FLinearColor | Streak tint |
| `bCAEnabled` | bool | CA enable |
| `CAStrength` | float | CA displacement |
| `bDistortionEnabled` | bool | Distortion enable |
| `DistortionK1` | float | k1 coefficient |
| `DistortionK2` | float | k2 coefficient |
