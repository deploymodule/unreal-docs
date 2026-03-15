# Aether — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Aether"` to your `Build.cs`.

## 2. Enable the Plugin

Open **Edit → Plugins**, search for **Aether**, enable it, and restart the editor.

## 3. Quick Enable

```ini
r.Aether.Enabled 1
r.Aether.BlendStrength 1.0
```

Or via Blueprint:

```cpp
UAetherSubsystem* Aether = GEngine->GetEngineSubsystem<UAetherSubsystem>();
Aether->SetEnabled(true);
Aether->SetBlendStrength(1.0f);
```

## 4. Adjust Stroke Character

The three most impactful parameters are kernel radius, sharpness, and orientation sensitivity:

```ini
; Large soft strokes (impressionist)
r.Aether.KernelRadius 8
r.Aether.Sharpness 4.0
r.Aether.OrientationSensitivity 1.0

; Fine detailed strokes (watercolor)
r.Aether.KernelRadius 4
r.Aether.Sharpness 8.0
r.Aether.OrientationSensitivity 2.0
```

## 5. Partial Blend (Subtle Stylization)

A blend of 0.3–0.5 gives a stylized-but-readable result that works well for stylized realistic games:

```cpp
Aether->SetBlendStrength(0.35f);
```

## 6. Animate a Stylization Wipe

Use Blueprint to animate the blend strength over time for a dramatic real-time transition from photorealistic to painterly:

```cpp
// In a tick function or timeline callback
float T = FMath::Clamp(ElapsedTime / TransitionDuration, 0.f, 1.f);
Aether->SetBlendStrength(FMath::SmoothStep(0.f, 1.f, T));
```

## 7. Performance Mode

```ini
; Run AKF at half resolution
r.Aether.ResScale 0.5

; Reduce sector count from 8 to 4 (faster, slightly less smooth)
r.Aether.SectorCount 4
```

## 8. Per-Volume Blending

Place an `AAetherPostProcessVolume` actor to apply the painterly effect only in certain areas (e.g., a dream sequence zone):

```cpp
AAetherPostProcessVolume* Vol = SpawnActor<AAetherPostProcessVolume>();
Vol->Settings.BlendStrength = 1.0f;
Vol->Settings.KernelRadius = 6.f;
Vol->BlendRadius = 200.f;  // world-space falloff
```
