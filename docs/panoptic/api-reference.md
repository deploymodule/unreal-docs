# Panoptic — API Reference

## APanopticVCamActor

`APanopticVCamActor` is the root actor class for all Panoptic functionality. Place one or more of these in the level instead of a standard `UCineCameraComponent`-bearing actor when using Panoptic.

| Property | Type | Description |
|----------|------|-------------|
| `bAutoOpenEditorPanel` | `bool` | When true, the Panoptic editor panel opens automatically when this actor is selected in the editor. |
| `DefaultModifierStack` | `TArray<TSubclassOf<UVCamModifier>>` | Modifier classes to add automatically when the actor is first placed in a level. Defaults to all ten Panoptic modifiers in recommended stack order. |
| `HUDWidgetClass` | `TSubclassOf<UUserWidget>` | Widget class to use for the HUD overlay. Override to provide a custom HUD layout. |
| `PositionModifier` | `TObjectPtr<UPanopticModifier_Positions>` | Direct reference to the Positions modifier instance on this actor, set automatically on BeginPlay. |
| `FocusModifier` | `TObjectPtr<UPanopticModifier_Focus>` | Direct reference to the Focus modifier instance, set automatically on BeginPlay. |

`APanopticVCamActor` also exposes Blueprint-callable helpers for toggling light rig groups and arming the countdown modifier, intended for use from a VCam input binding or a custom operator interface.

## UPanopticSettings

`UPanopticSettings` is a `UDeveloperSettings`-derived class stored in the project's `Config/DefaultPanoptic.ini`. Changes made in **Edit > Project Settings > Panoptic** are written here.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `SessionSaveRoot` | `FString` | `/Game/Panoptic/Sessions` | Content path where session log `.uasset` files are written by TakeManager. |
| `MaxPositionBookmarks` | `int32` | `8` | Number of camera position slots. Changing this above 8 requires a corresponding UI update in `SPanopticPositions`. |
| `DefaultFocusUpdateInterval` | `float` | `0.05` | Seconds between depth-sample updates for `UPanopticModifier_Focus`. |
| `DefaultTransitionDuration` | `float` | `1.0` | Default seconds for camera position recall transitions. |
| `LightFadeDuration` | `float` | `0.5` | Default fade duration in seconds for light rig toggle operations. |
| `WaveformMode` | `EPanopticWaveformMode` | `Luma` | Default waveform display mode for the HUD waveform widget. |
| `bAutoLoadLastSession` | `bool` | `true` | When true, TakeManager loads the most recent session `.uasset` from `SessionSaveRoot` on editor startup. |
| `bShowHUDByDefault` | `bool` | `true` | When true, all HUD widgets are visible when the VCam output opens. |

## Modifier Classes

| Class | Purpose | Key Properties |
|-------|---------|----------------|
| `UPanopticModifier_Focus` | Multi-mode autofocus with rack focus | `FocusMode` (`EFocusMode`), `TrackTarget` (`AActor*`), `FocusUpdateInterval` (`float`), `RackTargetDistance` (`float`), `RackDuration` (`float`), `FocusZones` (`TArray<FPanopticFocusZone>`) |
| `UPanopticModifier_ImageAssist` | Post-process monitoring overlays | `bEnableFocusPeaking` (`bool`), `bEnableFalseColor` (`bool`), `bEnableZebraStripes` (`bool`), `ZebraThreshold` (`float`) |
| `UPanopticModifier_LookLUT` | Look LUT applied to VCam output | `LUTTexture` (`UTexture2D*`), `LUTIntensity` (`float`) |
| `UPanopticModifier_LensDistortion` | Barrel/pincushion lens distortion | `DistortionAmount` (`float`), `bBarrel` (`bool`) |
| `UPanopticModifier_MotionProfile` | Named motion preset for UE movement modifiers | `MotionProfile` (`EMotionProfile`), `NoiseIntensity` (`float`), `DampingFactor` (`float`) |
| `UPanopticModifier_FollowTarget` | Smooth camera follow to a scene actor | `FollowTarget` (`AActor*`), `FollowOffset` (`FVector`), `FollowLag` (`float`), `RotationLag` (`float`) |
| `UPanopticModifier_AxisLock` | Lock individual translation/rotation axes | `bLockLocationX/Y/Z` (`bool`), `bLockRotationPitch/Yaw/Roll` (`bool`), `LockedValues` (`FTransform`) |
| `UPanopticModifier_DollyTrack` | Constrain camera to a spline path | `DollySpline` (`USplineComponent*`), `TrackPosition` (`float`, 0–1), `bAutoAdvance` (`bool`), `AdvanceSpeed` (`float`) |
| `UPanopticModifier_Positions` | Eight named bookmarks with smooth recall | `Bookmarks` (`TArray<FPanopticPositionBookmark>`), `TransitionDuration` (`float`), `TransitionCurve` (`UCurveFloat*`) |
| `UPanopticModifier_Countdown` | Countdown timer triggering TakeRecorder | `CountdownDuration` (`float`), `bArmOnBeginPlay` (`bool`), `OnCountdownComplete` (delegate) |

## Focus Modes

`EFocusMode` is declared in `PanopticFocusTypes.h` in the `Panoptic` module.

| Value | Description |
|-------|-------------|
| `Center` | Samples scene depth at the exact screen centre. |
| `Scatter` | Averages a configurable number of depth samples distributed across the frame. |
| `TrackActor` | Line-traces to the actor assigned to `TrackTarget`. Focus distance equals the trace hit distance. |
| `ZoneAverage` | Computes a weighted average depth across one or more user-defined `FPanopticFocusZone` screen-space rectangles. |

## Motion Profiles

`EMotionProfile` is declared in `PanopticMotionTypes.h` in the `Panoptic` module.

| Value | Description |
|-------|-------------|
| `Tripod` | No noise or lag. All axes held firm. Equivalent to disabling the movement modifier entirely. |
| `Steadicam` | Position and rotation lag with smooth damping. Simulates the floating response of a camera stabiliser rig. |
| `Handheld` | Low-amplitude Perlin noise applied to position and rotation each tick. Provides an organic micro-movement quality. |
| `Drone` | Physics-style damped response to input. Large position changes introduce inertia that bleeds off over several frames. |

## Waveform Modes

`EPanopticWaveformMode` is declared in `PanopticRenderingTypes.h` in the `PanopticRendering` module. It controls which channel the RDG compute shader displays in the waveform monitor HUD widget.

| Value | Description |
|-------|-------------|
| `Luma` | Displays the luminance (Y) channel of the VCam frame. Standard broadcast waveform. |
| `Red` | Displays the red channel only. |
| `Green` | Displays the green channel only. |
| `Blue` | Displays the blue channel only. |
| `Parade` | Displays all three colour channels side by side as a full waveform parade. |

## Light Rig Tags

These tag string constants are used to identify actors that belong to Panoptic's light rig. They are defined as `FName` literals in `PanopticLightRig.h`.

| Constant | Tag String | Usage |
|----------|-----------|-------|
| `PanopticTag_SceneLight` | `PanopticSceneLight` | Key lights and scene-motivated practicals. Toggled as a group by the **Scene Lights** control in the editor panel. |
| `PanopticTag_HouseLight` | `PanopticHouseLight` | Fill, ambient, and volume house lights. Toggled as a group by the **House Lights** control. |

Both tags can be present on the same actor if it should respond to both scene and house toggle operations.

!!! note
    Tags are read at toggle time via `UGameplayStatics::GetAllActorsWithTag`. There is no cached actor list — tag additions and removals are reflected immediately on the next toggle operation.
