# Tonal — API Reference

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Tonal.Enabled` | int | 0 | Master enable switch |
| `r.Tonal.Preset` | int | 0 | Built-in preset index (0–4) |
| `r.Tonal.FilmStrength` | float | 1.0 | Film stock curve blend (0=bypass) |
| `r.Tonal.Grain.Enabled` | int | 0 | Film grain enable |
| `r.Tonal.Grain.Intensity` | float | 0.04 | Grain intensity |
| `r.Tonal.Grain.Size` | float | 1.0 | Grain texel size multiplier |
| `r.Tonal.Halation.Enabled` | int | 0 | Halation enable |
| `r.Tonal.Halation.Threshold` | float | 0.8 | HDR luminance threshold for halation |
| `r.Tonal.Halation.Spread` | float | 12.0 | Blur radius in pixels |
| `r.Tonal.Halation.Intensity` | float | 0.25 | Halation blend strength |
| `r.Tonal.VHS.Enabled` | int | 0 | VHS mode enable |
| `r.Tonal.VHS.Intensity` | float | 0.5 | Overall VHS artifact intensity |
| `r.Tonal.VHS.DropoutRate` | float | 0.01 | Signal dropout frequency |
| `r.Tonal.LUT.Enabled` | int | 0 | Creative LUT enable |
| `r.Tonal.LUT.Strength` | float | 1.0 | LUT blend strength |

## UTonalSubsystem

`#include "TonalSubsystem.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetEnabled` | `void SetEnabled(bool bEnabled)` | Master on/off |
| `ApplyPreset` | `void ApplyPreset(ETonalPreset Preset)` | Load built-in film stock preset |
| `SetFilmStrength` | `void SetFilmStrength(float Strength)` | Film curve blend factor |
| `SetGrainEnabled` | `void SetGrainEnabled(bool bEnabled)` | Toggle grain stage |
| `SetGrainIntensity` | `void SetGrainIntensity(float Intensity)` | Grain intensity |
| `SetHalationEnabled` | `void SetHalationEnabled(bool bEnabled)` | Toggle halation stage |
| `SetHalationSpread` | `void SetHalationSpread(float Pixels)` | Halation blur radius |
| `SetHalationIntensity` | `void SetHalationIntensity(float Intensity)` | Halation strength |
| `SetVHSEnabled` | `void SetVHSEnabled(bool bEnabled)` | Toggle VHS stage |
| `SetVHSIntensity` | `void SetVHSIntensity(float Intensity)` | VHS artifact strength |
| `SetVHSDropoutRate` | `void SetVHSDropoutRate(float Rate)` | Dropout frequency |
| `SetLUT` | `void SetLUT(UTonalLUTAsset* LUT, float Strength)` | Assign and blend a LUT |
| `GetSettings` | `UTonalSettings* GetSettings()` | Mutable settings object |

## ETonalPreset

```cpp
UENUM(BlueprintType)
enum class ETonalPreset : uint8
{
    None            = 0,
    KodakVision3    = 1,
    FujiEterna500   = 2,
    Kodak2383Print  = 3,
    Monochrome      = 4,
};
```

## UTonalSettings

| Property | Type | Description |
|----------|------|-------------|
| `bEnabled` | bool | Master enable |
| `ActivePreset` | ETonalPreset | Current film stock preset |
| `FilmStrength` | float | Curve blend (0–1) |
| `GrainIntensity` | float | Grain strength |
| `GrainSize` | float | Grain size multiplier |
| `bHalationEnabled` | bool | Halation on/off |
| `HalationThreshold` | float | HDR threshold |
| `HalationSpread` | float | Blur radius |
| `HalationIntensity` | float | Halation strength |
| `bVHSEnabled` | bool | VHS mode on/off |
| `VHSIntensity` | float | VHS intensity |
| `LUTAsset` | UTonalLUTAsset* | Creative LUT reference |
| `LUTStrength` | float | LUT blend |
