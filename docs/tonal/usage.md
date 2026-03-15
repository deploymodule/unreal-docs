# Tonal — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Tonal"` to your `Build.cs`.

## 2. Enable the Plugin

Open **Edit → Plugins**, search for **Tonal**, enable it, and restart the editor.

## 3. Apply a Film Stock Preset

The fastest way to get started is to select a built-in preset via Blueprint:

```cpp
UTonalSubsystem* Tonal = GEngine->GetEngineSubsystem<UTonalSubsystem>();
Tonal->ApplyPreset(ETonalPreset::KodakVision3_500T);
```

Or via CVar:

```ini
r.Tonal.Preset 1
```

| Preset ID | Name |
|-----------|------|
| 0 | None (neutral) |
| 1 | Kodak Vision3 500T |
| 2 | Fuji Eterna 500 |
| 3 | Kodak 2383 Print |
| 4 | Monochrome |

## 4. Fine-Tune Parameters

```ini
; Film stock curve strength (0 = bypass, 1 = full effect)
r.Tonal.FilmStrength 0.85

; Film grain
r.Tonal.Grain.Enabled 1
r.Tonal.Grain.Intensity 0.04
r.Tonal.Grain.Size 1.5

; Halation
r.Tonal.Halation.Enabled 1
r.Tonal.Halation.Threshold 0.8
r.Tonal.Halation.Spread 12.0
r.Tonal.Halation.Intensity 0.3
```

## 5. VHS Mode

```cpp
Tonal->SetVHSEnabled(true);
Tonal->SetVHSIntensity(0.6f);
Tonal->SetVHSDropoutRate(0.02f);
```

!!! note
    VHS mode is designed for stylized/retro projects. Disable it for photorealistic scenes.

## 6. Custom LUT

1. Create a Data Asset of type `UTonalLUTAsset`
2. Assign your `.cube` file as the source texture
3. Apply via Blueprint:

```cpp
Tonal->SetLUT(MyLUTAsset, 0.8f); // 0.8 = blend strength
```

## 7. Sequencer Animation

Tonal parameters support Sequencer keyframing:

1. Open your Level Sequence
2. Add Track → **Tonal Track**
3. Keyframe any parameter — Film Strength, Grain Intensity, Halation, VHS, etc.

## 8. Post Process Volume Integration

Place a `ATonalPostProcessVolume` actor to apply localized grades (e.g., desaturate inside a cave).

```cpp
ATonalPostProcessVolume* Vol = SpawnActor<ATonalPostProcessVolume>();
Vol->Settings.FilmStrength = 0.5f;
Vol->Settings.bMonochrome = true;
```
