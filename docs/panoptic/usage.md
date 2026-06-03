# Panoptic — Usage

## Installation

1. Copy the `Panoptic` plugin folder into your project's `Plugins/` directory.
2. Open the project `.uproject` in a text editor and confirm `Panoptic` appears in the `Plugins` array with `"Enabled": true`, or enable it via **Edit > Plugins > Virtual Production**.
3. Recompile the project. Panoptic requires a C++ project; it cannot be used in a Blueprint-only project.
4. Restart the editor. The Panoptic editor panel will be available under **Window > Panoptic**.

!!! note
    Panoptic depends on the `VCam` plugin and on `ContinuityEditor`. Both must be enabled. If `ContinuityEditor` is not present, the `PanopticEditor` module will fail to load.

## Setting Up the VCam Actor

Panoptic's functionality lives on a `APanopticVCamActor` placed in the level.

1. In the **Place Actors** panel, search for `PanopticVCamActor` and drag it into the viewport.
2. Select the actor and open the **Details** panel.
3. Locate the **VCam Modifier Stack** section on the `UCineCameraComponent`.
4. Add the modifiers you need in the recommended stack order (see [Overview — VCam Modifier Stack](overview.md#vcam-modifier-stack)).

!!! warning
    Modifier stack order matters. Place `UPanopticModifier_Focus` before `UPanopticModifier_ImageAssist`, and place `UPanopticModifier_AxisLock` after `UPanopticModifier_FollowTarget`. Reversing these produces undefined interaction between modifier outputs.

Each modifier exposes its own properties in the Details panel when selected in the stack list. Configure each modifier before going live.

## Configuring Focus

1. Add `UPanopticModifier_Focus` to the modifier stack.
2. In the modifier's Details, set **Focus Mode** to one of: `Center`, `Scatter`, `TrackActor`, or `ZoneAverage`.
3. If using `TrackActor`, set the **Track Target** property to the actor you want the camera to lock focus on. The modifier will perform a line trace to that actor's root component each update cycle.
4. Adjust **Focus Update Interval** (seconds) to control how frequently depth is re-sampled. A value of `0.05` (20 Hz) is a reasonable starting point for most on-set use.
5. To use rack focus, set **Rack Target Distance** to the desired depth in centimetres and **Rack Duration** to the transition time in seconds. The rack begins on the next modifier update.

!!! tip
    `ZoneAverage` mode requires configuring at least one focus zone. Zones are defined as normalised screen-space rectangles with individual weights. Add zones in the modifier's Details panel under **Focus Zones**.

## Using Image Assist

1. Add `UPanopticModifier_ImageAssist` to the modifier stack, positioned after the Focus modifier.
2. The three overlays — focus peaking, false color, and zebra stripes — are each controlled by a boolean property on the modifier:
    - **Enable Focus Peaking**
    - **Enable False Color**
    - **Enable Zebra Stripes**
3. Toggle these from the modifier's Details panel, from the Panoptic editor panel, or from the HUD overlay directly using the operator controls.
4. Adjust **Zebra Threshold** (0.0–1.0 normalised luminance) to set the exposure level at which zebra stripes appear.

!!! note
    Image Assist overlays are purely monitoring aids. They affect only the VCam operator's view and have no effect on frames recorded by TakeRecorder.

## Saving Camera Positions

1. Add `UPanopticModifier_Positions` to the modifier stack.
2. Frame the shot you want to bookmark in the VCam viewport.
3. In the Panoptic editor panel, go to the **Positions** tab. Eight numbered slots are shown.
4. Click the slot you want to assign and press **Save Current**. The current world transform of the VCam actor is stored in that slot. Optionally type a name for the bookmark.
5. To recall a position, click the slot and press **Go To**. The modifier will interpolate the camera to the stored transform over the configured **Transition Duration**.

Transition Duration and Transition Curve are configured per-modifier in the Details panel. The default curve is ease-in/ease-out.

!!! tip
    Use the **Axis Lock** modifier in the stack after Positions to prevent the recall transition from drifting on axes you want held constant — for example, locking roll prevents any tilt introduced during a position move.

## Choosing a Motion Profile

1. Add `UPanopticModifier_MotionProfile` to the modifier stack.
2. Set **Motion Profile** in the modifier's Details to one of the four presets:

| Profile | Behaviour |
|---------|-----------|
| `Tripod` | No movement noise, all axes held firm. Use for locked-off shots. |
| `Steadicam` | Smooth lag on position and rotation with gentle damping. Mimics a real Steadicam rig's floating quality. |
| `Handheld` | Subtle Perlin noise on position and rotation axes. Adds organic micro-movement to a handheld aesthetic. |
| `Drone` | Damped physics-style response to position changes. Replicates the inertia of a drone in flight. |

3. Switch profiles at any time by changing the property. The transition is immediate.

## Recording Takes

### Choosing the REC Target Shot

Before recording, go to the **Shot Checklist** tab in the Panoptic editor panel. Each row corresponds to a shot from the Prevu plugin. Click the toggle in the `(*)` column of the shot you are about to record. The active REC-target shot name appears in the status bar next to the **REC** button so you can confirm the selection before triggering.

### Using the Countdown Modifier

1. Add `UPanopticModifier_Countdown` to the modifier stack (last in the stack is recommended).
2. Set **Countdown Duration** in seconds.
3. Press **Arm** in the editor panel or HUD. The countdown begins and is displayed in the HUD overlay. When the countdown reaches zero, TakeRecorder starts automatically.

### Manual Recording

Press the **REC** button in the Panoptic editor panel or trigger TakeRecorder directly via its own panel. Panoptic's TakeManager listens to the `OnRecordingStarted` delegate and stamps the active REC-target shot name onto the new take row automatically.

### After Recording

When recording stops (via `OnRecordingFinished` or `OnRecordingCancelled`), the session log is updated and saved. The take row in the **Take Log** tab will show the shot name, duration, and any flags set during the take.

## Reviewing the Session Log

The session log is saved as a `.uasset` file at `/Game/Panoptic/Sessions/` in your project's Content directory. The most recent session is loaded automatically when the editor starts.

To review a previous session:

1. Open the **Content Browser** and navigate to `Content/Panoptic/Sessions/`.
2. Double-click the session asset. Its contents will appear in the **Take Log** tab of the Panoptic panel.

Each take row in the log shows: take index, shot name, start timecode, duration, status (complete / cancelled), and any operator flags set during the take.

## Setting Up the Light Rig

Panoptic's light controls work through actor tags. No scripting or plugin-specific references are required.

1. Select a light actor in the viewport or **World Outliner**.
2. In the **Details** panel, locate the **Tags** array under the Actor section.
3. Add one of the following tags:

| Tag | Use For |
|-----|---------|
| `PanopticSceneLight` | Key lights, practicals, and any scene-motivated source |
| `PanopticHouseLight` | Fill, ambient, and volume space illumination |

4. Repeat for all lights that should be part of the rig.
5. In the Panoptic editor panel, use the **Lights** section to toggle scene lights, house lights, or all lights. Set a **Fade Duration** if you want a gradual transition rather than an instant toggle.

!!! tip
    Tags can be added or removed at any time during the session. The light rig controller reads tags at toggle time, so changes take effect immediately without restarting anything.

## Using the Editor Panel

Open the Panoptic panel via **Window > Panoptic**. The panel is dockable and can be resized like any other Unreal tab.

### Multi-Cam Preview Grid

The top section of the panel shows a grid of live previews for all `APanopticVCamActor` instances in the scene. Click any cell to make that camera the active VCam output. The active camera is highlighted with a border.

### Shot Checklist

The Shot Checklist tab mirrors the shot list from the Prevu plugin. Each row shows:

- Shot name and scene number
- A `(*)` / `( )` toggle indicating whether this shot is the current REC target
- A status indicator updated after each take

Set the REC target for the next recording by clicking the `(*)` column of the desired row. Only one shot can be the REC target at a time.

### Take Log

The Take Log tab shows the persistent session record. Rows are added in real time as takes complete. The log cannot be edited manually — it is a read-only audit trail of the session.

### Camera Positions

The Positions tab displays the eight bookmark slots. Each slot shows its assigned name (or "Empty" if not yet saved), a thumbnail of the stored view if available, and **Save Current** / **Go To** buttons.
