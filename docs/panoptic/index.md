# Panoptic

Virtual camera toolkit for solo filmmakers — image monitoring overlays, multi-mode autofocus, camera bookmarks, motion profiles, take management, and a director's HUD built on top of Unreal Engine's VCam system.

| | |
|---|---|
| **Category** | Virtual Production |
| **UE Version** | 5.7+ |
| **Target** | Runtime + Editor |
| **Intended Use** | On-set pre-production and production VCam work |

Panoptic extends the UE VCam stack with a suite of 10 evaluated modifiers and an editor panel designed for the demands of live virtual production. It is not a game-runtime tool — it is built for operators using VCam on set, where the director needs monitoring feedback, repeatable positions, a controlled recording pipeline, and quick light rig access without leaving Unreal.

## Features

- Ten VCam modifiers evaluated in a deterministic stack, covering focus, image assist, LUT, lens distortion, motion profiles, follow target, axis lock, dolly track, positions, and countdown
- Four autofocus modes — Center, Scatter, TrackActor, and ZoneAverage — with budget-throttled depth sampling
- Image monitoring overlays: focus peaking, false color, and zebra stripes toggled via MPC without interrupting the VCam output
- Eight named camera position bookmarks with configurable smooth transition curves
- Four motion profiles (Tripod, Steadicam, Handheld, Drone) that configure built-in UE movement modifiers
- TakeRecorder bridge with session log persisted as a `.uasset`, per-take shot correlation, and countdown trigger modifier
- Director's HUD overlay with 11 widgets including waveform monitor, exposure histogram, continuity PIP, and virtual slate
- Tag-based light rig control for scene and house lights with fade duration, requiring no hard actor references

## In This Section

- [Overview](overview.md) — Module layout, modifier stack, focus system, HUD, and take management architecture
- [Usage](usage.md) — Installation, modifier setup, recording pipeline, light rig tagging, and editor panel walkthrough
- [API Reference](api-reference.md) — Classes, enums, properties, and actor tag constants
- [Changelog](changelog.md) — Version history
