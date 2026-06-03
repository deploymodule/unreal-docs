# Prevu — Usage

## Installation

To enable the plugin, add it to your project's `.uproject` file under the `Plugins` array:

```json
{
  "Name": "Prevu",
  "Enabled": true
}
```

Then open the editor, go to **Edit > Plugins**, search for "Prevu", and ensure it is checked. Restart the editor when prompted.

!!! warning
    Prevu targets **Win64** and **UE 5.7**. The `Prevu` module is `UncookedOnly` — it will not be compiled into a packaged game. Do not place Prevu project assets (vocabulary data assets, shot templates, project files) anywhere in your content tree that feeds the cook. Use the Developer folder or mark them as Never Cook.

The plugin depends on the engine's built-in **Takes** plugin (`TakesCore` + `TakeRecorder`). Ensure it is enabled before opening Prevu.

## Opening Prevu

Once the plugin is enabled, open the main Prevu panel from:

**Window > Prevu**

The panel docks as a standard editor tab. You can float it, dock it beside the Content Browser, or embed it in a custom editor layout.

The panel consists of three regions:

| Region | Widget | Purpose |
|--------|--------|---------|
| Left | `SPrevuHierarchy` | Act/Scene/Shot/Step tree |
| Center | `SPrevuBoardView` | Thumbnail grid (storyboard view) |
| Right | `SPrevuShotInspector` | Duration, notes, tags, steps list for the selected shot |
| Top | `SPrevuToolbar` | Assemble, New Act/Scene/Shot, Export, Import |

## Creating Your First Project

When you open Prevu with no existing project, the panel shows an empty state. Use the toolbar buttons to build the hierarchy top-down:

1. Click **New Act** in the toolbar. A new Act appears in the hierarchy panel. Double-click to rename it (for example, "Act 1").
2. With the Act selected, click **New Scene**. Rename the scene to match your location or story beat.
3. With the Scene selected, click **New Shot**. Rename the shot (for example, "Shot 01 — Establishing Wide").
4. The new shot appears in both the hierarchy panel and the board view as an empty thumbnail placeholder.

Repeat for every shot in your production. You do not need to assemble sequences until you are ready to use them in Sequencer.

## Working in the Hierarchy Panel

The hierarchy panel (`SPrevuHierarchy`) shows the full Act → Scene → Shot → Step tree. Standard interactions:

- **Expand / collapse** an act or scene by clicking the disclosure arrow.
- **Select** any node to populate the shot inspector on the right and highlight the corresponding thumbnail in the board view.
- **Rename** any node by double-clicking its label.
- **Reorder shots** within a scene by dragging them up or down in the tree.
- **Add a Step** by selecting a shot and clicking **New Step** in the inspector panel's steps list. Steps appear as sub-items under the shot in the hierarchy.
- **Delete** any node by right-clicking and choosing Remove.

!!! note
    Reordering shots, adding steps, or renaming nodes updates only the Prevu data model. It does not modify any Level Sequence asset. The toolbar Assemble icon will tint amber to indicate that sequences are now stale. See [Assembling the Master Sequence](#assembling-the-master-sequence).

## Capturing Thumbnails

Thumbnails are captured using a `SceneCapture2D` component positioned to match the active camera in the level.

**Per-shot capture:**

1. Select the shot in the hierarchy or board view.
2. Position your camera in the viewport to the desired framing.
3. In the shot inspector, click **Capture Thumbnail**.
4. The thumbnail updates in the board view immediately.

**Batch capture:**

1. In the toolbar, open the Export/Capture menu.
2. Choose **Capture All Thumbnails** to iterate every shot in the project, or **Capture Scene** to capture only the selected scene's shots.
3. Prevu will step through each shot in sequence. Ensure the relevant level is open and cameras are placed before running a batch capture.

Thumbnail images are stored at paths recorded on each `FPrevuShot` and referenced by the storyboard export system.

## Taking Level Snapshots

A Level Snapshot records the state of the current level and associates it with a specific **Step** under a shot. Snapshots let you save multiple layout or blocking variants non-destructively and restore any of them later.

**Taking a snapshot:**

1. Arrange actors in the level to the desired state for this step.
2. In the hierarchy panel, select or create the Step you want to record.
3. In the shot inspector's steps list, click **Take Snapshot** next to the step.
4. Prevu calls `PrevuSnapshotManager::TakeSnapshot`, which writes a Level Snapshot asset and stores its reference on `FPrevuStep`.

**Restoring a snapshot:**

1. Select the Step in the hierarchy or inspector.
2. Click **Restore** next to the step.
3. `PrevuSnapshotManager::RestoreSnapshot` applies the snapshot using `PrevuRestoreFilter`, which limits the restore to only the actors and properties that Prevu manages. Other level content is unaffected.

!!! tip
    Snapshots are most useful when blocking out complex scenes with multiple camera positions or prop arrangements. Save a step for each variant so you can compare them side by side without re-staging.

## Assembling the Master Sequence

Assembling is the explicit step that creates or rebuilds the actual Level Sequence assets from the Prevu data model. You must assemble before using the master sequence in Sequencer, before rendering via Continuity, or before passing it to any other downstream tool.

**When to assemble:**

- After creating or reordering shots
- After changing shot durations
- After any structural change to the hierarchy

The toolbar's Assemble button tints **amber** whenever the data model has changed since the last assemble. This is your signal that sequences are stale.

**How to assemble:**

1. Click the **Assemble** button in the toolbar (amber when stale, normal color when up to date).
2. Prevu's `PrevuSequencerBridge::AssembleMasterSequence` runs. It constructs the full Master → Act → Scene → Shot Level Sequence nesting, converting shot durations from display frames to tick resolution using `DisplayToTick`.
3. When complete, the amber tint clears and all Level Sequence assets reflect the current data model.

!!! warning
    Do not skip the assemble step before using the master sequence downstream. The sequence assets are only as current as your last assemble. The amber icon is the authoritative indicator of staleness.

## Inspecting a Shot

Select any shot in the hierarchy or board view to populate the inspector panel (`SPrevuShotInspector`) on the right.

| Field | Description |
|-------|-------------|
| Duration | Shot length in frames at the project frame rate. Changing this marks sequences as stale. |
| Notes | Free-text field for director notes, action descriptions, or lens information. |
| Tags | Comma-separated or multi-select tags for filtering and export grouping. |
| Steps list | All steps under this shot, each with a thumbnail preview, label, snapshot status, and Restore button. |
| Audio clip | Reference to the scratch audio recorded for this shot. |
| Level Sequence | Reference to the shot's backing `ULevelSequence` asset (populated after assemble). |

## Exporting a Storyboard

Prevu supports three export formats. Open the export dialog from the **Export** button in the toolbar.

**PNG storyboard sheet:**

Exports a flat image compositing all shot thumbnails with frame numbers, shot names, duration, and notes. Use this for printing, sharing via email, or embedding in a production PDF.

**HTML storyboard sheet:**

Exports a self-contained HTML file with the same content as the PNG sheet. The file has no external dependencies and can be opened in any browser. Useful for sharing with collaborators who do not have Unreal installed.

**`.prevu` JSON bundle:**

Exports the entire `UPrevuProject` data model as a JSON file with the `.prevu` extension. The bundle includes all acts, scenes, shots, steps, metadata, notes, tags, and relative asset references. It can be checked into version control, shared without Unreal, and re-imported into any Prevu installation via the **Import** toolbar button.

!!! note
    The `.prevu` bundle stores references to binary assets (thumbnails, Level Snapshots) but does not embed them. When sharing a bundle with collaborators, also share the referenced asset files or re-capture thumbnails after import.
