# Panoptic — Overview

## What It Is

Panoptic is a virtual camera toolkit built on top of Unreal Engine's VCam system. It targets solo filmmakers and small crews doing live virtual production — pre-production scouts, on-set VCam operator sessions, and director reviews. The plugin adds professional image monitoring tools (focus peaking, false color, zebra stripes, waveform, histogram), multi-mode autofocus, repeatable camera positions with smooth transitions, motion profile presets, a TakeRecorder bridge with persistent session logging, quick light rig control, and a director's HUD overlay rendered on the VCam output.

Panoptic is not a game-runtime system. It is designed for use inside the Unreal Editor with VCam active. The `Panoptic` and `PanopticRendering` modules are declared as Runtime-type only because `UVCamModifier` and `UActorComponent` subclasses must live in Runtime modules for VCam to evaluate them — the plugin has no meaningful role in a packaged build.

## Module Layout

| Module | Type | Description |
|--------|------|-------------|
| `Panoptic` | Runtime | `APanopticVCamActor`, `UPanopticSettings`, `FPanopticModule`, all ten `UPanopticModifier_*` classes |
| `PanopticRendering` | Runtime | RDG compute shaders for the waveform monitor, `EPanopticWaveformMode` enum |
| `PanopticEditor` | Editor | `FPanopticEditorModule`, all Slate UI (`SPanoptic*`), TakeManager, ContinuityBridge, ExposureMatch, HUDManager, SelectionBridge, ActorRegistry |

`PanopticEditor` has a hard compile dependency on `ContinuityEditor` for shot name and range queries via `IContinuityShotProvider`.

## VCam Modifier Stack

VCam evaluates modifiers in the order they appear in the modifier stack on the `UCineCameraComponent`. Panoptic's ten modifiers are designed to be added in the order listed below; placing them out of order can cause unexpected interactions (for example, applying AxisLock before FollowTarget may suppress motion that FollowTarget intends to introduce).

| Modifier | Purpose |
|----------|---------|
| `UPanopticModifier_Focus` | Multi-mode autofocus with budget-throttled depth sampling and rack focus |
| `UPanopticModifier_ImageAssist` | Post-process monitoring overlays: focus peaking, false color, zebra stripes |
| `UPanopticModifier_LookLUT` | Applies a user-specified look LUT to the VCam output |
| `UPanopticModifier_LensDistortion` | Barrel or pincushion lens distortion via post-process |
| `UPanopticModifier_MotionProfile` | Configures built-in UE movement modifiers with named motion presets |
| `UPanopticModifier_FollowTarget` | Smooth camera follow locked to a tracked scene actor |
| `UPanopticModifier_AxisLock` | Locks individual translation or rotation axes to prevent unwanted drift |
| `UPanopticModifier_DollyTrack` | Constrains camera movement to a spline path |
| `UPanopticModifier_Positions` | Eight named bookmarks with smooth position recall transitions |
| `UPanopticModifier_Countdown` | Countdown timer that triggers a cued TakeRecorder recording |

## Focus System

`UPanopticModifier_Focus` implements four autofocus modes selectable at runtime from the editor panel or HUD.

### Modes

| Mode | Behaviour |
|------|-----------|
| `Center` | Reads scene depth at the exact center of the screen. Fast and reliable for centered subjects. |
| `Scatter` | Averages multiple depth samples distributed across the frame. More stable for subjects that do not stay centered. |
| `TrackActor` | Performs a line trace to a specified scene actor. Provides consistent, subject-locked focus independent of depth buffer variance. |
| `ZoneAverage` | Computes a weighted average across configurable screen zones. Useful for wide shots where different zones should contribute unequally. |

### Budget Throttling

Depth reads are expensive under VCam frame budgets. The focus modifier exposes a `FocusUpdateInterval` property that limits how frequently depth samples are re-evaluated. On frames where the budget is not due, the modifier reuses the previous focus distance rather than skipping the evaluation entirely, so focus distance always has a valid value.

### Rack Focus

The modifier supports a manual rack-focus target: set a target depth value and a rack duration and the focus distance will animate smoothly from the current value to the target over the specified time, then hold.

## Image Assist

`UPanopticModifier_ImageAssist` adds three professional monitoring overlays to the VCam output. Each overlay is toggled by enabling or disabling a parameter in a Material Parameter Collection (`MPC_Panoptic`) rather than by adding or removing post-process materials, which avoids any hitch when toggling.

| Overlay | What It Shows |
|---------|--------------|
| Focus Peaking | Highlights edges in the frame that are within the depth-of-field range, coloured for visibility. Useful for verifying focus pull accuracy on a VCam feed. |
| False Color | Replaces the luminance channel with a colour ramp keyed to exposure stops. Helps the operator confirm exposure without reading numbers. |
| Zebra Stripes | Overlays animated diagonal stripes on pixels above a configurable luminance threshold. Classic broadcast overexposure indicator. |

All three overlays can be active simultaneously. They are purely visual and have no effect on captured output from TakeRecorder.

## Camera Positions

`UPanopticModifier_Positions` stores up to eight named camera bookmarks. Each bookmark records the full world transform of the VCam actor at the time the bookmark is saved.

Recalling a bookmark does not teleport the camera. Instead, the modifier drives a smooth interpolation from the current transform to the stored transform over a configurable transition duration. The transition curve defaults to an ease-in/ease-out but can be set to linear or a custom curve asset.

`UPanopticModifier_AxisLock` works alongside the positions system. When a lock is active on any axis (for example, locking the roll axis to prevent unwanted tilt), the lock is applied after other modifiers run, so position transitions still evaluate but the locked axes are clamped before output.

## HUD Overlay

Panoptic renders eleven HUD widgets on the VCam output. Visibility of each widget is managed by `HUDManager` in the editor module and can be toggled individually.

| Widget | Content |
|--------|---------|
| Lens Data | Current focal length (mm), aperture (f-stop), and focus distance |
| Frame Guides | Safe area boundary, rule-of-thirds grid, and center crosshair |
| Recording Indicator | Flashing REC indicator and elapsed time when TakeRecorder is active |
| Timecode | Editor timecode read from the engine timecode provider |
| Continuity PIP | Picture-in-picture thumbnail of the most recent captured shot from the Continuity plugin |
| DoF Helper | Depth-of-field visualisation showing near/far focus planes |
| Sequencer Info | Active sequence name and current playhead frame |
| Shot Metadata | Shot name and scene/take numbers sourced from the Continuity bridge |
| Virtual Slate | Clapper slate graphic populated with shot, scene, take, and camera fields |
| Waveform Monitor | RDG compute-shader waveform rendered via `PanopticRendering`, mode selectable via `EPanopticWaveformMode` |
| Exposure Histogram | Luminance histogram of the current VCam frame |

## Take Management

`TakeManager` in `PanopticEditor` bridges Unreal's TakeRecorder with Panoptic's shot checklist and session log.

### Delegate Binding

Rather than polling `UTakeRecorderBlueprintLibrary::IsRecording()` each tick, TakeManager binds the three TakeRecorder lifecycle delegates at startup:

- `OnRecordingStarted` — stamps the current shot name from the Continuity bridge onto the new take row
- `OnRecordingFinished` — marks the take complete, flags any issues set via the HUD, and saves the session log
- `OnRecordingCancelled` — marks the take as cancelled and saves the log

This delegate model ensures takes are logged correctly even when recording is triggered by the default countdown path rather than a direct UI button press.

### Session Log Persistence

The session log is a `UObject`-derived asset saved via `CreatePackage` and `SavePackage` into the project's `/Game/Panoptic/Sessions/` path. On editor startup, TakeManager loads the most recently modified session asset so the log survives editor restarts. The log is written after every take and on editor shutdown.

### Shot Correlation

The shot checklist in `SPanopticShotChecklist` shows shots sourced from the Prevu plugin. Each row has a REC-target toggle (`(*)/( )`) that the operator sets to indicate which shot the next recording should be stamped with. The active REC-target shot name is shown next to the REC button so there is no ambiguity before triggering a take.

## Light Rig

Panoptic uses actor tags rather than hard references for light rig control. Any light actor in the scene can be tagged with one of two tags:

| Tag | Meaning |
|-----|---------|
| `PanopticSceneLight` | Key lights and scene-motivated practical lights |
| `PanopticHouseLight` | Ambient and fill lights that illuminate the volume space |

The light rig controller iterates over all tagged actors at toggle time rather than maintaining a cached list. This means lights can be added to or removed from the rig at any point by editing their tags in the Details panel — no plugin restart or reconfiguration required.

Toggle operations accept a fade duration parameter. When a non-zero duration is provided, the controller animates the intensity of each affected light from its current value to zero (or back) over that duration rather than snapping.

Scene lights, house lights, or all lights can be toggled independently.
