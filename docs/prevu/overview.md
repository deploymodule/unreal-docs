# Prevu — Overview

## What It Is

Prevu is an editor-only C++/Slate plugin that brings storyboard and animatic authoring into Unreal Engine. The core problem it solves is that conventional pre-production tools — storyboard software, animatics editors, scratch audio apps — all live outside the engine, forcing a constant translation step between a director's intent and the Sequencer-based production timeline.

Prevu eliminates that gap. All shot planning, thumbnail capture, scratch audio recording, and animatic assembly happen inside the editor, against the actual scene geometry and lighting. The resulting master sequence is a real `ULevelSequence` that can be handed directly to a Continuity-managed shoot or a Sequencer-based cinematic pipeline.

Because Prevu is an authoring tool, it ships as `UncookedOnly` and does not appear in a packaged game. Assets must be kept out of the cook path (Developer folder or Never Cook settings) to satisfy this guarantee.

## Module Layout

| Module | Type | Description |
|--------|------|-------------|
| `Prevu` | `UncookedOnly` | Data model: `UPrevuProject`, struct types, Blueprint library, vocabulary data asset, shot template |
| `PrevuEditor` | `Editor` | All Slate UI panels, sequencer bridge, capture, snapshot, audio, import/export systems |

The split ensures that the data model types are accessible from Blueprint and from other editor plugins (such as Panoptic's shot checklist), while all editor-specific Slate and engine-editor dependencies are isolated in `PrevuEditor`.

## Hierarchy Model

Prevu structures a production as four nested levels:

```
UPrevuProject
  └─ FPrevuAct           (e.g. "Act 1", "Prologue")
       └─ FPrevuScene    (e.g. "Scene 3 — Rooftop")
            └─ FPrevuShot   (e.g. "Shot 04 — Wide")
                 └─ FPrevuStep   (e.g. "Step A — Camera push in")
```

| Level | Purpose |
|-------|---------|
| Act | Largest narrative division. Corresponds to a top-level sub-sequence under the master. |
| Scene | A location or continuous block within an act. Gets its own sub-sequence nested under the act sequence. |
| Shot | The primary unit of authoring. Stores duration, notes, tags, thumbnail, scratch audio, and a Level Sequence reference. |
| Step | A named stage within a shot — used for on-set iteration or blocking variants. Each step stores its own thumbnail and a Level Snapshot reference so the scene state can be restored selectively. |

## Sync Model

This is the most important thing to understand about Prevu: **editing data in the UI does not touch Level Sequences.**

When you create, rename, reorder, or delete acts, scenes, shots, or steps, Prevu updates only its own internal data model and refreshes its Slate views (the hierarchy panel, board view, and shot inspector). No `ULevelSequence` asset is created, modified, or deleted during these operations.

The only operation that rebuilds Level Sequence assets is **Assemble Master Sequence**, triggered by the Assemble button in the toolbar. Assemble is an explicit "bake" step — it walks the entire data model and constructs the full Master → Act → Scene → Shot nesting from scratch.

The consequence is that between assembles, the Level Sequence assets may not reflect the current state of the data model. This is intentional: it lets you iterate quickly on structure and timing without waiting for Sequencer writes on every edit.

To make this visible, `UPrevuProject` carries a transient staleness flag. Any mutation of the data model calls `MarkSequencesOutOfDate()`. Assemble calls `ClearSequencesOutOfDate()`. While the flag is set, the Assemble button in the toolbar tints **amber** and shows a "re-assemble" tooltip. When sequences are up to date the button returns to its normal color.

!!! warning
    Sequences are stale by design between assembles. Do not rely on Level Sequence assets reflecting shot timing or order until you have clicked Assemble after your last edit.

!!! tip
    The amber toolbar icon is your signal. If it is amber, click Assemble before using the master sequence in Sequencer, Continuity, or any downstream tool.

## Sequencer Assembly

`PrevuSequencerBridge::AssembleMasterSequence` is the single entry point for all Level Sequence construction. It performs the following steps in order:

1. Creates or locates the master `ULevelSequence` asset for the project.
2. For each Act in the data model, creates or locates an act sub-sequence and nests it in the master via a `UMovieSceneSubSection`.
3. For each Scene within that Act, creates or locates a scene sub-sequence nested under the act.
4. For each Shot within that Scene, creates or locates a shot sequence nested under the scene, and sets its playback range from the shot's duration.

A critical detail is the frame number conversion. Unreal's Sequencer stores all frame positions at the sequence's **tick resolution** (typically 24000 ticks per second), not at display frames. A shot with a duration of 48 display frames at 24 fps must be stored as `48 * (24000 / 24) = 48000` ticks. The `DisplayToTick` helper function performs this conversion and is applied at every `AddSequence` site. Without it, assembled sequences appear to have the correct playback range in the editor but are approximately 1000x shorter than intended.

`ClearSequencesOutOfDate()` is called after a successful assemble, which removes the amber tint from the toolbar.

## Captures and Snapshots

**Thumbnail capture** is handled by `PrevuCapture`. It places a `SceneCapture2D` actor in the world, positions it according to the active camera, triggers a capture, and writes the result to a path stored on `FPrevuShot`. Captures can be triggered per-shot from the inspector panel or in batch across all shots in a scene or the full project. The thumbnail path is also used in storyboard sheet exports.

**Level Snapshots** are managed by `PrevuSnapshotManager`. A snapshot records the state of the current level — actor transforms, component properties, and other serializable state — and associates the snapshot asset with a specific `FPrevuStep`. Restoring a snapshot returns the level to that state, using `PrevuRestoreFilter` to limit the restore to only the actors and properties that Prevu cares about, avoiding unintended side-effects on unrelated level content.

The per-step snapshot workflow supports on-set blocking iteration: you can save multiple layout variants as separate steps under one shot and switch between them non-destructively.

## Export Formats

Prevu supports three export formats, all driven by `PrevuImportExport` and `PrevuExporter`:

| Format | Description |
|--------|-------------|
| PNG storyboard sheet | A flat image compositing all shot thumbnails with frame numbers, shot names, duration, and notes. Suitable for printing or dropping into a PDF deck. |
| HTML storyboard sheet | A self-contained HTML file with the same content as the PNG sheet, but interactive. No external dependencies. |
| `.prevu` JSON bundle | A complete serialization of the `UPrevuProject` data model including all metadata, notes, tags, and relative asset references. Portable — can be checked into version control, shared without Unreal, and re-imported into any Prevu installation. |

!!! note
    The `.prevu` bundle does not embed binary assets such as thumbnail images or Level Snapshots. It stores references to them. Ensure those assets are included when distributing the bundle to collaborators.
