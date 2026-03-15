# Vapor — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Vapor"` to your `Build.cs` dependencies.

## 2. Enable the Plugin

Open **Edit → Plugins**, search for **Vapor**, enable it, and restart the editor.

## 3. Place a Post Process Volume

Add a **Post Process Volume** to your level. Enable **Infinite Extent (Unbound)** for a global effect, or set a bounded volume for a localized fog zone.

## 4. Configure via Detail Panel

Select the Post Process Volume. In the **Details** panel, scroll to the **Vapor** category:

| Property | Default | Description |
|----------|---------|-------------|
| Enable Fog Scattering | false | Master on/off switch |
| Density | 0.3 | Overall fog density |
| Height Falloff | 0.5 | How quickly fog thins with altitude |
| Scattering Color | (1, 0.97, 0.93) | Tint of scattered light |
| Phase G | 0.6 | Henyey-Greenstein anisotropy (0=isotropic, 1=full forward) |
| Sky Glow Intensity | 1.0 | Contribution of sky dome to in-scatter |

## 5. CVars (Console / INI)

All properties map to CVars. Use the console (`~`) or `DefaultEngine.ini`:

```ini
[/Script/Engine.RendererSettings]
r.Vapor.Enabled=1
r.Vapor.Density=0.3
r.Vapor.HeightFalloff=0.5
r.Vapor.PhaseG=0.6
r.Vapor.SkyGlowIntensity=1.0
r.Vapor.ResolutionScale=0.5
r.Vapor.RaymarchSteps=32
```

## 6. Blueprint API

```cpp
// Get the subsystem
UVaporSubsystem* Vapor = GEngine->GetEngineSubsystem<UVaporSubsystem>();

// Adjust parameters at runtime
Vapor->SetDensity(0.5f);
Vapor->SetScatterColor(FLinearColor(1.f, 0.95f, 0.88f));
Vapor->SetPhaseG(0.7f);
Vapor->SetEnabled(true);
```

Blueprint nodes are available under the **Vapor** category in the node search.

## 7. Additional Light Sources

Vapor can incorporate up to 8 point/spot lights into the scattering computation. Register a light via Blueprint:

```cpp
Vapor->RegisterScatterLight(MyPointLight);
Vapor->UnregisterScatterLight(MyPointLight);
```

Registered lights are uploaded to the constant buffer each frame; unregistered lights cost nothing.

## 8. Fog Volumes (Localized Fog)

Place an `AVaporFogVolume` actor in the level. The volume defines a box region with its own density override. Multiple volumes blend additively.

!!! warning "Performance"
    Each additional fog volume adds a masked scatter pass. Keep fog volumes to 4 or fewer for smooth frame times.
