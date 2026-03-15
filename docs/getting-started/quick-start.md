# Quick Start

A five-minute walkthrough to get your first PluginDepot plugin running.

## Example: Vapor (Fog Scattering)

### 1. Install the Plugin

Follow the [Installation](installation.md) guide for the Vapor plugin.

### 2. Place a Post Process Volume

Add a **Post Process Volume** to your level and enable **Infinite Extent**.

### 3. Enable Vapor

In the Post Process Volume details, find the **Vapor** category and enable **Fog Scattering**.

### 4. Tweak Parameters

Adjust density, scattering color, and light contribution from the detail panel — changes are reflected in real time via CVars.

```ini
; Or use CVars directly in the console
r.Vapor.Density 0.5
r.Vapor.ScatterColor 1.0 0.95 0.9
```

### 5. Blueprint API

```blueprint
// Get the Vapor subsystem and adjust at runtime
VaporSubsystem->SetDensity(0.75f);
```

---

!!! note "Other Plugins"
    Each plugin's own [Usage](../vapor/usage.md) page has a tailored quick-start example.
