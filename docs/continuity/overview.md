# Continuity — Overview

## What It Is

Continuity is an editor-only plugin that layers a complete production workflow on top of UE5's native Sequencer. It does not fork or replace Sequencer — every feature attaches through official extension points: `ISequencerModule` for toolbar and filter-bar injection, `UToolMenus` for menu entries, and Slate docking for the Sequence Browser panel. This means Continuity upgrades any project that uses Level Sequences for cinematic work without changing how Sequencer itself operates.

The plugin has no runtime component. It compiles as a single editor module (`ContinuityEditor`) and contributes nothing to packaged builds. All configuration lives in Project Settings, and all persistent state is either stored in the Level Sequence assets themselves or written to disk as sidecar files alongside rendered output.

## Module Layout

Continuity ships as one module.

| Module | Type | Description |
|--------|------|-------------|
| `ContinuityEditor` | Editor | All features — browser, toolbar, filters, handles, exporters |

The module loads at `PostEngineInit` phase so that Sequencer and `UToolMenus` are both fully initialized before Continuity registers its extensions. There is no separate runtime or client module.

## Sequence Browser

The Sequence Browser is a dockable Slate panel that reads the currently open level's Level Sequences and presents them in a three-level tree: Act, Scene, Shot. The hierarchy is derived entirely from each sequence's asset name using delimiter and token patterns configured in **Project Settings > Plugins > Continuity > Naming Conventions**.

For example, with the default pattern `ACT{A}_SC{SS}_SH{SSS}`:

- `ACT1` groups all sequences whose names begin with `ACT1`
- `ACT1_SC01` groups all shots in that scene
- `ACT1_SC01_SH010` is a leaf shot node

Each row in the tree displays the sequence's duration in seconds, its total frame count at the sequence's tick resolution, and in/out points if working range markers have been set. Double-clicking any row opens that sequence in the active Sequencer tab. Right-clicking exposes context actions including Rename (which updates the asset name and propagates the token change), Duplicate, and Add to Render Queue.

!!! note
    The browser reflects the level's world. It does not scan the Content Browser. Sequences must be present as level sub-sequences or as independent assets associated with the open level to appear in the tree.

## Toolbar Extensions

Continuity injects a contiguous group of buttons into Sequencer's native toolbar. All buttons are registered via `ISequencerModule::RegisterToolBarExtension` and respect the native toolbar's layout so they dock flush with Sequencer's own controls.

| Button | Action |
|--------|--------|
| Mark In | Sets the working range start to the current playhead position |
| Mark Out | Sets the working range end to the current playhead position |
| Lock Range | Toggles the playback range lock so scrubbing cannot move markers |
| Jump to Camera Cut (Prev) | Moves the playhead to the previous camera cut section boundary |
| Jump to Camera Cut (Next) | Moves the playhead to the next camera cut section boundary |
| Collapse All | Collapses all expanded track rows in the Sequencer panel |
| Expand All | Expands all collapsed track rows |
| Isolate | Hides all tracks except those currently selected; toggle again to restore |
| Auto-Size | Opens a submenu with Fit Parent to Children and Fit Children to Parent |
| Show Handles | Toggles the visual display of handle overlap regions on all sections |
| Render Shot | Queues the sequence currently open in Sequencer as a Movie Render Queue job using the selected Render Pass Preset |
| Timecode | A read-only display showing the current playhead position as HH:MM:SS:FF |

## Handle Manager

Continuity's Handle Manager implements what the codebase calls Option C: when handles are applied to a shot, it extends **both** the inner sequence's working range **and** the parent sub-sequence section's overlap region by the configured handle duration. This ensures that the parent timeline sees the pre-roll and post-roll frames without any gap or clip at the cut boundary.

Handle duration is set in frames at **Project Settings > Plugins > Continuity > Handle Manager > Handle Frames**. The default is 8 frames. Changing this value does not retroactively update existing sequences — use the Apply Handles toolbar action to update all shots in the current hierarchy.

The Show Handles toolbar button toggles a colored overlay on every sub-sequence section that makes the handle region (the overlap beyond the shot's cut-in and cut-out) visible as a distinct color band. This makes it easy to audit which shots have handles applied and which do not.

## Render Pipeline

### Batch Shot Queuer

The Batch Shot Queuer scans the sequence hierarchy and creates one Movie Render Queue (MRQ) job per shot. It does not reimplement rendering — it constructs `UMoviePipelineQueue` jobs and hands them to MRQ for execution. Output paths use naming-convention tokens so the rendered files land in an organized folder structure that mirrors the ACT/SCENE/SHOT hierarchy.

Selecting shots before opening the queue dialog restricts queueing to the selection. Without a selection, all shots in the hierarchy are queued.

### Render Pass Preset DataAssets

Each Render Pass Preset is a `UContinuityRenderPassPreset` DataAsset that configures the MRQ settings for a specific delivery scenario.

| Preset | Intended Use |
|--------|-------------|
| Editorial | Fast proxy encode, JPEG compression, half resolution — for internal review cuts |
| Colorist Package | Full EXR sequence with embedded LUT reference — for color grading delivery |
| Full CG | EXR with additional passes: ambient occlusion, cryptomatte, diffuse, specular — for VFX work |
| Proxy | Low-resolution H.264, no alpha — for lightweight review and offline editing |

You can create additional Render Pass Preset DataAssets in the Content Browser to define custom pipelines.

## Export Formats

### EDL (CMX 3600)

The EDL Exporter writes a CMX 3600 Edit Decision List that describes the assembled cut as a sequence of edit events. Each shot in the hierarchy becomes one event. Timecode values are derived from the sequence's frame rate and working range. Reel names are derived from the shot's naming-convention tokens.

The resulting `.edl` file is suitable for direct import into DaVinci Resolve for color grading or conform, and into most professional NLEs for timeline reconstruction.

### CSV Production Sheets

The CSV Exporter writes a flat production sheet with one row per shot. Four profiles are available:

| Profile | Columns |
|---------|---------|
| Master | Shot name, duration, frame range, camera, notes, status |
| Colorist | Shot name, frame range, camera, LUT, color notes, status |
| VFX | Shot name, frame range, camera, VFX pass names, complexity, status |
| ShotGrid | Shot name, duration, frame range, camera, ShotGrid-compatible status field |

!!! warning
    CSV export is write-only. Continuity does not import CSV data back into the project. Use ShotGrid, Google Sheets, or your production tracking tool to manage the status columns after export.

### Shot Sidecar JSON

When a shot is rendered through the Batch Shot Queuer, Continuity writes a `.continuity.json` sidecar file alongside the rendered EXR frames. The sidecar records the shot name, frame range, camera actor name, the list of enabled render pass names, and an ISO 8601 timestamp of the render. This file is intended for pipeline scripts that need to correlate rendered files back to their Unreal source sequences.

## Integration with Other Plugins

Continuity exposes `IContinuityShotProvider`, a lightweight `IModularFeatures` interface that other plugins can query to get information about the currently active shot without taking a hard compile-time dependency on Continuity.

Panoptic (`PanopticEditor` module) registers a dependency on this interface at startup to populate its monitoring overlay with the current shot name and frame range. If Continuity is not present, Panoptic falls back to showing no shot information — it does not crash.

See [API Reference — IContinuityShotProvider](api-reference.md#icontinuityshotprovider) for function signatures.
