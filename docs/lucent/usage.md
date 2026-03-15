# Lucent — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Lucent"` to your `Build.cs`.

## 2. Enable the Plugin

Open **Edit → Plugins**, search for **Lucent**, enable it, and restart the editor.

## 3. Configure Depth of Field

```cpp
ULucentSubsystem* Lucent = GEngine->GetEngineSubsystem<ULucentSubsystem>();

// Physical lens parameters
Lucent->SetFocalLength(85.f);       // mm
Lucent->SetAperture(1.8f);          // f-stop
Lucent->SetFocusDistance(400.f);    // cm (to subject)
Lucent->SetSensorWidth(36.f);       // mm (full-frame = 36)
```

Or via CVars:

```ini
r.Lucent.DOF.Enabled=1
r.Lucent.DOF.FocalLength=85
r.Lucent.DOF.Aperture=1.8
r.Lucent.DOF.FocusDistance=400
r.Lucent.DOF.SensorWidth=36
```

## 4. Custom Bokeh Shape

1. Import a 128×128 grayscale texture as a `ULucentBokehAsset`
2. Assign it to the Lucent settings:

```cpp
Lucent->SetBokehShape(MyBokehAsset);
```

## 5. Bloom

```ini
r.Lucent.Bloom.Enabled=1
r.Lucent.Bloom.Intensity=0.5
r.Lucent.Bloom.Threshold=1.0
; Per-scale tints (6 scales)
r.Lucent.Bloom.Scale0Tint=1.0 0.95 0.9
r.Lucent.Bloom.Scale5Tint=0.8 0.6 0.4
```

## 6. Anamorphic Streaks

```cpp
Lucent->SetStreaksEnabled(true);
Lucent->SetStreakLength(120.f);         // pixels
Lucent->SetStreakColor(FLinearColor(0.4f, 0.7f, 1.f));
Lucent->SetStreakCount(ELucentStreakCount::Two);
```

## 7. Chromatic Aberration

```ini
r.Lucent.CA.Enabled=1
r.Lucent.CA.Strength=0.004
r.Lucent.CA.RadialFalloff=2.0
```

## 8. Barrel Distortion

```ini
r.Lucent.Distortion.Enabled=1
r.Lucent.Distortion.K1=-0.12
r.Lucent.Distortion.K2=0.05
```

## 9. Sequencer Focus Pull

1. Open your Level Sequence
2. Add Track → **Lucent Track**
3. Add `FocusDistance` channel
4. Keyframe from 200cm to 600cm for a smooth focus pull

The Lucent sequencer track drives CVars directly — no gameplay-thread lag.

## 10. Focus on Actor

```cpp
// Auto-focus on a specific actor (raycasts each frame)
Lucent->SetAutoFocusTarget(MyCharacter);
Lucent->SetAutoFocusSpeed(8.f);  // interpolation speed
```
