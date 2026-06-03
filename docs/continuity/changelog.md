# Continuity — Changelog

## v0.1.0 — Initial Release

### Added

- `ContinuityEditor` single-module plugin structure (Editor type, no runtime component) loading at `PostEngineInit` phase to ensure safe registration with `UToolMenus` and `ISequencerModule`
- **Sequence Browser** — dockable Slate panel that reads all Level Sequences in the open level and organizes them as a three-level ACT/SCENE/SHOT hierarchy derived from configurable naming convention patterns
- Sequence Browser displays per-shot duration, frame count, and in/out working range markers on each shot row
- Double-click to open any shot in the active Sequencer tab; right-click context menu with Open, Add to Render Queue, Rename, and Duplicate actions
- Manual Refresh button to rebuild the browser tree after adding sequences to the level
- **Sequencer Toolbar Extensions** registered via `ISequencerModule::RegisterToolBarExtension`: Mark In, Mark Out, Lock Range, Jump to Camera Cut (Prev/Next), Collapse All, Expand All, Isolate, Auto-Size submenu, Show Handles, Render Shot button, and timecode display
- **Track Type Filter** groups injected into Sequencer's native filter bar: Camera, Transform, Properties, Audio, Events, and Custom categories, each toggling visibility of matching track types simultaneously
- **Handle Manager** implementing Option C: extends both the inner sequence's working range and the parent sub-sequence section's overlap region by the configured handle frame count
- Handle Frames setting in Project Settings (default: 8 frames); Apply Handles batch action available from the toolbar dropdown and Sequence Browser root context menu
- Show Handles toggle that overlays a colored band on every sub-sequence section's handle region for visual auditing
- **Auto-Size** with two operations: Fit Parent to Children (expands/shrinks parent working range to contain all child sections) and Fit Children to Parent (trims/extends all child sections to fill the parent range), both wrapped in `FScopedTransaction` for undo support
- **Batch Shot Queuer** that scans the sequence hierarchy and creates one `UMoviePipelineQueue` job per shot in Movie Render Queue, with support for queuing all shots or only browser-selected shots
- Output path token expansion in the Batch Shot Queuer using naming-convention tokens: `{ACT}`, `{SCENE}`, `{SHOT}`, `{PASS}`, `{FRAME}`, `{DATE}`, and `{PROJECT}`
- **Render Pass Preset DataAsset** (`UContinuityRenderPassPreset`) system for defining named MRQ configurations; four built-in presets shipped as DataAssets: Editorial, Colorist Package, Full CG, and Proxy
- Support for custom `UContinuityRenderPassPreset` DataAssets created in the Content Browser, appearing in all preset dropdowns without an editor restart
- **EDL Exporter** (`FContinuityEDLExporter`) writing CMX 3600 format from a master Level Sequence, with one edit event per sub-sequence section, timecode values derived from sequence frame rate and working range, and reel names from shot naming-convention tokens
- EDL export accessible via **Sequencer > Continuity > Export EDL** main menu entry
- `FContinuityEDLExportOptions` with flags for including disabled shots, using short reel names, and applying a start timecode offset
- **CSV Exporter** (`FContinuityCSVExporter`) with four profile modes: Master, Colorist, VFX, and ShotGrid
- CSV export accessible via **Sequencer > Continuity > Export CSV** with profile selection dropdown
- `EContinuityCSVProfile` enum covering all four profiles with distinct column sets per profile
- `FContinuityCSVExportOptions` with configurable header row, disabled-shot inclusion, and custom delimiter character
- **Shot Sidecar JSON** written automatically alongside rendered EXR frames by the Batch Shot Queuer, recording shot name, frame range, camera actor name, render pass names, preset name, and ISO 8601 render timestamp
- **Cinematic Viewport Extension** adding a shot name label overlay and a "Jump to Parent" button in the cinematic viewport when a sub-sequence shot is open in Sequencer
- **`IContinuityShotProvider`** IModularFeatures interface exposing `GetCurrentShotName`, `GetCurrentShotRange`, `GetCurrentSequence`, `GetCurrentCamera`, and `OnShotChanged` delegate for inter-plugin queries without hard dependencies
- Panoptic integration support: `IContinuityShotProvider` consumed by `PanopticEditor` to populate the monitoring overlay with current shot metadata
- **Project Settings page** at **Edit > Project Settings > Plugins > Continuity** covering Naming Conventions (token delimiter and three token patterns), Handle Manager (handle frame count and overlay color), Sequence Browser (auto-refresh, frame count display, in/out marker display), and Export (default EDL path, default CSV path, default Render Pass Preset)
- All settings serialized to `DefaultEngine.ini` under the `[/Script/ContinuityEditor.ContinuitySettings]` section
- `LogContinuity` log category for all plugin diagnostic output
- Full undo/redo support for all sequence-modifying operations via `FScopedTransaction`
