# Continuity — API Reference

## IContinuityShotProvider

`IContinuityShotProvider` is an `IModularFeatures` interface registered by `ContinuityEditor` at startup. It allows other modules to query the currently active shot without a hard compile-time dependency on Continuity.

`#include "IContinuityShotProvider.h"`

Retrieve the interface at runtime:

```cpp
IContinuityShotProvider* Provider = nullptr;
if (IModularFeatures::Get().IsModularFeatureAvailable("ContinuityShotProvider"))
{
    Provider = &IModularFeatures::Get().GetModularFeature<IContinuityShotProvider>(
        "ContinuityShotProvider");
}
```

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `GetCurrentShotName` | `FString GetCurrentShotName() const` | Returns the asset name of the sequence currently open in Sequencer, or an empty string if no shot is active |
| `GetCurrentShotRange` | `TRange<FFrameNumber> GetCurrentShotRange() const` | Returns the working range of the active sequence as a frame number range at the sequence's display rate |
| `GetCurrentSequence` | `ULevelSequence* GetCurrentSequence() const` | Returns a pointer to the active `ULevelSequence` asset, or `nullptr` if none is open |
| `GetCurrentCamera` | `FString GetCurrentCamera() const` | Returns the name of the camera actor bound in the active shot's Camera Cuts track, or an empty string if unbound |
| `OnShotChanged` | `FSimpleMulticastDelegate& OnShotChanged()` | Multicast delegate broadcast whenever the active shot changes (open, close, or switch) |

!!! note
    All functions are safe to call from the game thread only. `GetCurrentSequence` returns a raw pointer — do not store it across frames without a `TWeakObjectPtr`.

## Render Pass Preset DataAsset

`UContinuityRenderPassPreset` is a `UDataAsset` subclass. Create instances in the Content Browser under **Miscellaneous > Data Asset > ContinuityRenderPassPreset**.

`#include "ContinuityRenderPassPreset.h"`

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `PresetName` | `FText` | Display name shown in the Render Pass Preset dropdown in the toolbar and Queue dialog |
| `MRQPreset` | `UMoviePipelinePrimaryConfig*` | Reference to a Movie Render Queue primary config asset. All MRQ settings (output format, resolution, AA samples) are inherited from this asset |
| `OutputPathPattern` | `FString` | Token pattern for the rendered output path. Supports tokens `{ACT}`, `{SCENE}`, `{SHOT}`, `{PASS}`, `{FRAME}`, and any token defined in the Naming Conventions settings |
| `bEnabled` | `bool` | When false, the preset is hidden from all dropdowns. Defaults to true |
| `Description` | `FText` | Optional description displayed as a tooltip in the preset dropdown |

### Output Path Token Reference

| Token | Expands To |
|-------|-----------|
| `{ACT}` | Act identifier extracted from the shot name |
| `{SCENE}` | Scene identifier extracted from the shot name |
| `{SHOT}` | Full shot name (the Level Sequence asset name) |
| `{PASS}` | Render pass name as defined in the MRQ primary config |
| `{FRAME}` | Zero-padded frame number, e.g. `0042` |
| `{DATE}` | ISO 8601 date of the render, e.g. `2026-06-03` |
| `{PROJECT}` | Unreal project name |

## Project Settings

All settings live at **Edit > Project Settings > Plugins > Continuity**.

`#include "ContinuitySettings.h"` — accessible via `GetDefault<UContinuitySettings>()`.

### Naming Conventions

| Setting | Type | Description |
|---------|------|-------------|
| `TokenDelimiter` | `FString` | Character separating hierarchy tokens in sequence asset names. Default: `_` |
| `ActTokenPattern` | `FString` | Regex pattern matching the Act portion of a name, e.g. `ACT\d+` |
| `SceneTokenPattern` | `FString` | Regex pattern matching the Scene portion, e.g. `SC\d+` |
| `ShotTokenPattern` | `FString` | Regex pattern matching the Shot portion, e.g. `SH\d+` |

### Handle Manager

| Setting | Type | Description |
|---------|------|-------------|
| `HandleFrames` | `int32` | Number of handle frames to extend on each side of a shot. Default: 8 |
| `HandleOverlayColor` | `FLinearColor` | Color of the handle region overlay when Show Handles is enabled. Default: semi-transparent amber |

### Sequence Browser

| Setting | Type | Description |
|---------|------|-------------|
| `bAutoRefreshOnLevelLoad` | `bool` | Automatically rebuild the browser tree when a new level is opened. Default: true |
| `bShowFrameCount` | `bool` | Display frame count alongside duration in shot rows. Default: true |
| `bShowInOutMarkers` | `bool` | Display working range in/out values in shot rows. Default: true |

### Export

| Setting | Type | Description |
|---------|------|-------------|
| `DefaultEDLExportPath` | `FDirectoryPath` | Default directory offered in the EDL save dialog. Default: project's `Saved/Export` folder |
| `DefaultCSVExportPath` | `FDirectoryPath` | Default directory offered in the CSV save dialog. Default: project's `Saved/Export` folder |
| `DefaultRenderPassPreset` | `TSoftObjectPtr<UContinuityRenderPassPreset>` | Preset selected by default in the Queue dialog. Can be left null to require explicit selection |

## EDL Exporter

`#include "ContinuityEDLExporter.h"`

### ExportEDL

```cpp
bool FContinuityEDLExporter::ExportEDL(
    ULevelSequence*       MasterSequence,
    const FString&        OutputFilePath,
    const FFrameRate&     OutputFrameRate,
    FContinuityEDLExportOptions Options = FContinuityEDLExportOptions()
);
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `MasterSequence` | `ULevelSequence*` | The top-level sequence whose sub-sequence sections define the cut. Must not be null. |
| `OutputFilePath` | `const FString&` | Absolute path including filename and `.edl` extension |
| `OutputFrameRate` | `const FFrameRate&` | Frame rate used for all timecode values in the output. Typically the master sequence's display rate |
| `Options` | `FContinuityEDLExportOptions` | Optional settings struct (see below) |

Returns `true` on success. On failure, writes a reason to the output log under the `LogContinuity` category.

### FContinuityEDLExportOptions

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `bIncludeDisabledShots` | `bool` | `false` | Include sub-sequence sections that are muted or disabled in Sequencer |
| `bUseShortReelNames` | `bool` | `false` | Use only the Shot token as the reel name instead of the full sequence name |
| `StartTimecode` | `FTimecode` | `00:00:00:00` | Timecode offset applied to all record timecode values in the EDL |

## CSV Exporter

`#include "ContinuityCSVExporter.h"`

### ExportCSV

```cpp
bool FContinuityCSVExporter::ExportCSV(
    ULevelSequence*           MasterSequence,
    const FString&            OutputFilePath,
    EContinuityCSVProfile     Profile,
    FContinuityCSVExportOptions Options = FContinuityCSVExportOptions()
);
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `MasterSequence` | `ULevelSequence*` | The sequence whose shot hierarchy is enumerated for the sheet |
| `OutputFilePath` | `const FString&` | Absolute path with `.csv` extension |
| `Profile` | `EContinuityCSVProfile` | Which column set to write (see enum below) |
| `Options` | `FContinuityCSVExportOptions` | Optional settings struct |

Returns `true` on success.

### EContinuityCSVProfile

| Enum Value | Description |
|-----------|-------------|
| `EContinuityCSVProfile::Master` | All shots with shot name, duration, frame range, camera, notes, and status columns |
| `EContinuityCSVProfile::Colorist` | Shots with color-grade columns: shot name, frame range, camera, LUT reference, color notes, status |
| `EContinuityCSVProfile::VFX` | Shots with VFX columns: shot name, frame range, camera, VFX pass names, complexity rating, status |
| `EContinuityCSVProfile::ShotGrid` | ShotGrid-compatible layout: shot name, duration, frame range, camera, and a ShotGrid-compatible status field using ShotGrid's status code vocabulary |

### FContinuityCSVExportOptions

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `bIncludeHeader` | `bool` | `true` | Write a header row as the first line of the file |
| `bIncludeDisabledShots` | `bool` | `false` | Include muted or disabled shots in the sheet |
| `Delimiter` | `TCHAR` | `','` | Column delimiter character. Use `'\t'` for TSV output |

## Shot Sidecar

Shot sidecar files are written automatically by the Batch Shot Queuer when rendering completes. The file is named `<ShotName>.continuity.json` and placed in the same directory as the rendered EXR frames.

### Sidecar JSON Schema

```cpp
// Example sidecar file content
{
    "ShotName":      "ACT1_SC02_SH030",
    "FrameRangeIn":  1001,
    "FrameRangeOut": 1064,
    "Camera":        "CineCamera_Hero",
    "RenderPasses":  ["Beauty", "Depth", "Cryptomatte"],
    "RenderPreset":  "Full CG",
    "Timestamp":     "2026-06-03T14:22:01Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `ShotName` | string | Full Level Sequence asset name |
| `FrameRangeIn` | integer | First rendered frame number (inclusive) |
| `FrameRangeOut` | integer | Last rendered frame number (inclusive) |
| `Camera` | string | Actor name of the bound CineCameraActor in the Camera Cuts track |
| `RenderPasses` | string array | Names of all output passes in the MRQ job |
| `RenderPreset` | string | Display name of the `UContinuityRenderPassPreset` used |
| `Timestamp` | string | ISO 8601 UTC timestamp of render completion |
