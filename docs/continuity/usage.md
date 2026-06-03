# Continuity — Usage

## Installation

Follow the [Installation](../getting-started/installation.md) guide for the plugin distribution. To enable Continuity in your project:

1. Copy the `Continuity` folder into your project's `Plugins/` directory.
2. Open **Edit > Plugins**, search for **Continuity**, and enable it.
3. Restart the editor.

Continuity's module loads at the `PostEngineInit` phase. This is intentional — `UToolMenus` and `ISequencerModule` must be fully initialized before toolbar and filter bar extensions can be registered. Do not attempt to load the module earlier in your own `Build.cs` rules.

To add a compile-time dependency from another editor module (such as Panoptic):

```cpp
// YourModule.Build.cs
PrivateDependencyModuleNames.AddRange(new string[]
{
    "ContinuityEditor"
});
```

!!! note
    If you only need the `IContinuityShotProvider` interface and want to keep the dependency optional, use `IModularFeatures::Get().GetModularFeature<IContinuityShotProvider>` with a null check rather than a hard dependency. Panoptic uses this pattern.

## Configuring Naming Conventions

All hierarchy parsing is driven by the naming convention settings at **Edit > Project Settings > Plugins > Continuity > Naming Conventions**.

| Setting | What It Controls |
|---------|-----------------|
| Act Token Pattern | Regex or prefix pattern that identifies the Act portion of a sequence name |
| Scene Token Pattern | Regex or prefix pattern for Scene |
| Shot Token Pattern | Regex or prefix pattern for Shot |
| Token Delimiter | Character separating tokens in sequence names (default: `_`) |

For example, if your sequences are named `A01_S03_SH0050`, set the delimiter to `_` and configure the Act, Scene, and Shot patterns to match the `A01`, `S03`, and `SH0050` tokens respectively.

!!! tip
    Make naming convention decisions before creating sequences. The Sequence Browser derives hierarchy purely from names — renaming sequences later is safe but must be done through the browser's Rename action, not through the Content Browser, to keep parent sub-sequence references valid.

## Opening the Sequence Browser

Open the Sequence Browser from the main menu: **Window > Continuity > Sequence Browser**. The panel is dockable and can be placed alongside the Content Browser or Level Editor Outliner.

Alternatively, if the Sequencer toolbar is visible, the browser icon (a film-strip list) opens the panel directly.

The browser populates when a level is fully loaded. If sequences are added to the level while the browser is open, click **Refresh** at the top of the panel to rebuild the tree.

## Navigating Sequences

The browser displays the ACT/SCENE/SHOT hierarchy as an expandable tree. Each level shows:

- **Act row** — act identifier, total child sequence count
- **Scene row** — scene identifier, total shot count, combined duration
- **Shot row** — shot name, duration in seconds, frame range (start–end at sequence tick resolution), in/out working range markers

Click the disclosure triangle on an Act or Scene row to expand it. Double-click any Shot row to open that sequence in Sequencer. The Sequencer tab title updates to reflect the opened shot's full name.

Right-clicking a Shot row shows a context menu with:
- **Open in Sequencer** — same as double-click
- **Add to Render Queue** — queues this shot using the currently selected Render Pass Preset
- **Rename** — renames the sequence asset and updates all parent sub-sequence section references
- **Duplicate** — duplicates the Level Sequence asset and inserts a new shot row

## Using the Toolbar Extensions

The Continuity toolbar group appears at the right end of Sequencer's native toolbar after the editor restart.

**Working Range**

- **Mark In** — with a sequence open, click this to set the working range start to the current playhead frame. Equivalent to pressing `I` in Sequencer but always visible.
- **Mark Out** — sets the working range end. Equivalent to `O`.
- **Lock Range** — prevents the working range from being moved by scrubbing or playback. The button appears highlighted when the range is locked.

**Navigation**

- **Jump Prev Cut** / **Jump Next Cut** — moves the playhead to the nearest camera cut boundary in the Camera Cuts track. If no Camera Cuts track exists, the buttons are grayed out.

**Track Management**

- **Collapse All** / **Expand All** — collapses or expands every track row. Useful before exporting or taking a screenshot of the sequence.
- **Isolate** — hides all tracks except those currently selected. Click Isolate again to restore all hidden tracks. Isolation state is cleared when you close the sequence.

**Auto-Size**

Click **Auto-Size** to open a dropdown with two operations:

- **Fit Parent to Children** — expands or shrinks the currently open parent sequence's working range so it exactly contains all sub-sequence sections. No children are moved.
- **Fit Children to Parent** — trims or extends all sub-sequence sections to fill the parent's working range exactly. Children are repositioned at the working range start if they are outside it.

Both operations are wrapped in `FScopedTransaction` and appear in the undo history as a single action.

**Rendering**

- **Show Handles** — toggles the handle overlay. When enabled, each sub-sequence section shows a colored band on its leading and trailing edges representing the handle frames that extend beyond the cut boundary.
- **Render Shot** — queues the currently open sequence as an MRQ job. Before clicking, select a Render Pass Preset from the dropdown next to the button.

## Managing Handles

Handle duration is set globally at **Project Settings > Plugins > Continuity > Handle Manager > Handle Frames**. The default is 8 frames.

To apply handles to all shots in the hierarchy:

1. Open the master or parent sequence in Sequencer.
2. Click **Apply Handles** from the Continuity toolbar dropdown (or right-click the root Act row in the Sequence Browser).

Continuity extends both the inner sequence's working range (pre-roll and post-roll frames become part of the playable range) and the parent sub-sequence section's overlap region (the section in the parent timeline is stretched by the handle duration on each side).

Use **Show Handles** to audit the results. Sections that have not had handles applied will show no color band on their edges.

!!! warning
    Changing Handle Frames in Project Settings does not retroactively update existing sequences. You must re-run Apply Handles after changing the value.

## Filtering Track Types

The Track Type Filter adds a set of grouped category buttons to Sequencer's native filter bar. Click a category button to toggle the visibility of all Sequencer tracks that belong to that category.

| Category | Track Types Hidden / Shown |
|----------|--------------------------|
| Camera | CameraCut, CineCameraActor, FocusDistance, Aperture |
| Transform | TransformTrack, AttachTrack, PathTrack |
| Properties | PropertyTrack (all UProperty-bound tracks) |
| Audio | AudioTrack |
| Events | EventTrack, BlueprintEventTrack |
| Custom | Any tracks registered by third-party plugins that do not fit other categories |

Toggling a category applies immediately and does not modify the sequence asset. The filter state persists per editor session but resets when the editor is restarted.

## Batch Queuing Renders

To queue multiple shots for rendering:

1. Open the master sequence in Sequencer or select shots in the Sequence Browser.
2. From the Continuity toolbar, click the **Render Shot** dropdown and choose **Queue All Shots** (or **Queue Selected Shots** if shots are selected in the browser).
3. In the dialog that opens, choose a **Render Pass Preset** from the dropdown. Available presets are any `UContinuityRenderPassPreset` DataAssets in your project.
4. Review the output path preview, which shows how the naming-convention tokens will expand for each shot.
5. Click **Add to Queue**. Continuity creates one MRQ job per shot and opens the Movie Render Queue window.
6. In MRQ, click **Render (Local)** or **Render (Remote)** as normal.

!!! tip
    You can create custom Render Pass Preset DataAssets in the Content Browser under **Miscellaneous > Data Asset > ContinuityRenderPassPreset**. Custom presets appear in the dropdown immediately without requiring an editor restart.

## Exporting EDL

The EDL exporter produces a CMX 3600 `.edl` file from the assembled cut. Use this for DaVinci Resolve conform or NLE import.

1. Open the top-level master sequence (the sequence that contains all shots as sub-sequences) in Sequencer.
2. From the main menu, choose **Sequencer > Continuity > Export EDL**.
3. In the Save dialog, choose a destination and filename. The default filename uses the master sequence's name with a `.edl` extension.
4. Click **Save**.

The exporter writes one edit event per shot sub-sequence. Each event contains:
- Event number (sequential, starting at 001)
- Reel name derived from the shot's naming-convention tokens
- Source in/out timecode at the sequence's frame rate
- Record in/out timecode in the master cut

Import the resulting `.edl` into DaVinci Resolve via **File > Import Timeline > Import EDL**.

!!! note
    The EDL exporter uses the master sequence's frame rate for all timecode values. If individual shots have different tick resolutions, they are resampled to the master rate during export.

## Exporting CSV

The CSV exporter writes a flat production sheet suitable for import into ShotGrid, Google Sheets, or any spreadsheet tool.

1. From the main menu, choose **Sequencer > Continuity > Export CSV**.
2. Choose a **Profile** from the dropdown: Master, Colorist, VFX, or ShotGrid.
3. Click **Export** and choose a save location.

Each profile writes a different column set. The file is UTF-8 encoded with a header row. Open it in Excel or Google Sheets, or import directly into ShotGrid using the ShotGrid CSV import wizard.

!!! warning
    Continuity does not read CSV files back. The exported file is a snapshot of the current sequence state. Any status or notes columns you fill in externally are not reflected in the Unreal project.
