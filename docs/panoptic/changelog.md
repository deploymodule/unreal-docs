# Panoptic — Changelog

## v0.1.0 — Initial Release

### Added

- `APanopticVCamActor` — root actor class extending UE's VCam actor, serving as the placement target for all Panoptic systems. Exposes direct references to key modifier instances after BeginPlay.
- `UPanopticSettings` — developer settings class integrated with the Unreal Project Settings panel. Covers session save path, default focus update interval, default transition duration, light fade duration, waveform mode, and HUD visibility defaults.
- `FPanopticModule` (Runtime) and `FPanopticEditorModule` (Editor) — module entry points with lifecycle registration for all plugin systems.
- `PanopticRendering` module — second Runtime module housing RDG compute shaders for the waveform monitor. Kept separate from the main `Panoptic` module to isolate rendering dependencies.
- `UPanopticModifier_Focus` — four autofocus modes (Center, Scatter, TrackActor, ZoneAverage), rack focus with configurable target depth and duration, budget-throttled depth sampling via `FocusUpdateInterval`, and per-zone weight support for ZoneAverage mode.
- `UPanopticModifier_ImageAssist` — focus peaking, false color, and zebra stripe overlays toggled via Material Parameter Collection parameters on `MPC_Panoptic`. All three overlays can be active simultaneously without hitching.
- `UPanopticModifier_LookLUT` — applies a user-assigned 2D LUT texture to the VCam post-process chain with a blendable intensity parameter.
- `UPanopticModifier_LensDistortion` — barrel and pincushion lens distortion applied via post-process material, with a signed distortion amount property.
- `UPanopticModifier_MotionProfile` — four named presets (Tripod, Steadicam, Handheld, Drone) that configure underlying UE movement modifier parameters. Profile can be switched at runtime with immediate effect.
- `UPanopticModifier_FollowTarget` — smooth camera follow locked to a specified scene actor, with separate lag values for position and rotation.
- `UPanopticModifier_AxisLock` — per-axis lock for all six transform axes (Location X/Y/Z, Rotation Pitch/Yaw/Roll), applied as the final step in the modifier stack to suppress unwanted drift.
- `UPanopticModifier_DollyTrack` — constrains camera position to a spline component, with manual track position control and optional auto-advance with configurable speed.
- `UPanopticModifier_Positions` — eight named camera position bookmarks storing full world transforms. Recall triggers a smooth interpolation using a configurable duration and optional `UCurveFloat` asset.
- `UPanopticModifier_Countdown` — countdown timer that triggers TakeRecorder recording when the countdown reaches zero. Supports an arm-on-BeginPlay option and exposes an `OnCountdownComplete` delegate for custom integrations.
- `EPanopticWaveformMode` enum — Luma, Red, Green, Blue, Parade — controlling which channel the waveform monitor RDG compute shader renders.
- `EFocusMode` enum — Center, Scatter, TrackActor, ZoneAverage.
- `EMotionProfile` enum — Tripod, Steadicam, Handheld, Drone.
- `FPanopticFocusZone` struct — normalised screen-space rectangle with a weight float, used by ZoneAverage focus mode.
- `FPanopticPositionBookmark` struct — world transform, display name string, and optional thumbnail texture for a position slot.
- HUD overlay system with eleven widgets rendered on the VCam output: Lens Data, Frame Guides, Recording Indicator, Timecode, Continuity PIP, DoF Helper, Sequencer Info, Shot Metadata, Virtual Slate, Waveform Monitor, Exposure Histogram.
- `HUDManager` editor system — tracks widget visibility state, provides per-widget toggle API, and restores visibility configuration from project settings on session start.
- `TakeManager` editor system — binds `OnRecordingStarted`, `OnRecordingFinished`, and `OnRecordingCancelled` delegates from TakeRecorder. Stamps each take with the active REC-target shot name from the Continuity bridge. Saves the session log as a `.uasset` via `CreatePackage` + `SavePackage` after every take and on editor shutdown.
- Session log asset persistence — most recent session loaded automatically from `SessionSaveRoot` on editor startup via `UPanopticSettings::bAutoLoadLastSession`.
- `ContinuityBridge` editor system — queries Continuity's `IContinuityShotProvider` via `IModularFeatures` to retrieve the current shot name and frame range. Hard compile dependency on `ContinuityEditor`.
- `ExposureMatch` editor system — matches exposure settings between two selected cameras.
- `SelectionBridge` editor system — synchronises the actor selected in the editor's viewport with the active VCam actor tracked by Panoptic.
- `ActorRegistry` editor system — tracks scene actors for use by the Focus modifier's TrackActor mode and the FollowTarget modifier.
- `SPanopticPanel` — root dockable editor tab registered under **Window > Panoptic**, containing multi-cam preview grid, shot checklist, take log, and camera positions tabs.
- `SPanopticCamGrid` — multi-cam preview grid showing live thumbnails for all `APanopticVCamActor` instances in the scene. Click a cell to make that camera the active VCam output.
- `SPanopticShotChecklist` — shot list sourced from Prevu with per-row REC-target toggle (`(*)/( )`) and active-shot readout next to the REC button.
- `SPanopticTakeLog` — read-only take log view showing take index, shot name, start timecode, duration, and status for each logged take in the current session.
- `SPanopticPositions` — eight-slot camera bookmark panel with **Save Current** and **Go To** buttons and per-slot name labels.
- Tag-based light rig — `PanopticSceneLight` and `PanopticHouseLight` actor tags with independent and combined toggle controls, fade duration support, and no hard actor references.
- Plugin renamed from `Iris` / `IrisCam` to `Panoptic` to avoid API macro collision with UE's built-in `Iris` replication plugin. `CoreRedirects` added for all previous class, struct, and enum names.
