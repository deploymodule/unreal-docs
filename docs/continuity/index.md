# Continuity

Production Sequencer extension — shot browser, handle management, batch rendering, EDL/CSV export, and toolbar integration for UE5 filmmaking pipelines.

| | |
|---|---|
| **Category** | Virtual Production |
| **UE Version** | 5.7+ |
| **Target** | Editor only |
| **Runtime Impact** | None |
| **Module** | ContinuityEditor |

Continuity extends UE5's native Sequencer with a complete production layer without replacing or forking any engine code. It injects a dockable Sequence Browser, toolbar controls, and export actions directly into the editor via `UToolMenus` and `ISequencerModule`. A single editor module covers everything from first cut to final EDL delivery.

## Features

- Dockable Sequence Browser that reads the open level's sequences and displays them as an ACT/SCENE/SHOT hierarchy derived from configurable naming conventions
- Sequencer toolbar buttons for marking working ranges, locking playback, jumping between camera cuts, collapsing tracks, isolating selections, and queuing renders
- Track Type Filter groups that toggle whole categories of Sequencer tracks (Camera, Transform, Properties, Audio, Events, Custom) in one click
- Handle Manager that extends both the inner sequence's working range and the parent section's overlap region simultaneously (Option C)
- Auto-Size operations that fit a parent sequence's range to its children or fit children to fill the parent range, both with full undo support
- Batch Shot Queuer that scans the sequence hierarchy and creates Movie Render Queue jobs for every shot or a selected subset, driven by configurable output path tokens and Render Pass Preset DataAssets
- EDL Exporter that writes CMX 3600 format for DaVinci Resolve round-trip, deriving reel names from shot naming conventions
- CSV Exporter with four production profiles (Master, Colorist, VFX, ShotGrid) for downstream scheduling and color/VFX tracking
- Shot Sidecar JSON written alongside rendered EXRs, embedding shot name, frame range, camera, render pass names, and timestamp
- `IContinuityShotProvider` IModularFeatures interface for Panoptic and other plugins to query the currently active shot

UE 5.7 · Editor only · Virtual Production

---

- [Overview](overview.md) — Architecture, Sequence Browser, toolbar extensions, handles, render pipeline, and export formats
- [Usage](usage.md) — Step-by-step workflows for every major feature
- [API Reference](api-reference.md) — IContinuityShotProvider, Project Settings, Render Pass Preset DataAsset, and exporter signatures
- [Changelog](changelog.md) — Version history
