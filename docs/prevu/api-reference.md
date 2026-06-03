# Prevu — API Reference

This page documents the public-facing types and functions in the `Prevu` and `PrevuEditor` modules. All types live in the `Prevu` module unless noted as `PrevuEditor`.

## Data Model Types

The following table summarizes the five core types that compose a Prevu project.

| Type | Module | Kind | Description |
|------|--------|------|-------------|
| `UPrevuProject` | `Prevu` | `UObject` | Root asset. Owns the full Act array and project-level settings. Carries the sequences-out-of-date flag. |
| `FPrevuAct` | `Prevu` | Struct | Named act. Owns a `TArray<FPrevuScene>`. Corresponds to one act sub-sequence after assemble. |
| `FPrevuScene` | `Prevu` | Struct | Named scene within an act. Owns a `TArray<FPrevuShot>`. Corresponds to one scene sub-sequence. |
| `FPrevuShot` | `Prevu` | Struct | Primary authoring unit. Stores duration, notes, tags, thumbnail path, scratch audio ref, and Level Sequence ref. Owns a `TArray<FPrevuStep>`. |
| `FPrevuStep` | `Prevu` | Struct | Named stage within a shot. Stores label, thumbnail path, and Level Snapshot asset reference. |

### FPrevuShot — Key Fields

| Field | Type | Description |
|-------|------|-------------|
| `Name` | `FString` | Display name shown in hierarchy and board view |
| `DurationFrames` | `int32` | Shot length in display frames at the project frame rate |
| `Notes` | `FString` | Director or production notes |
| `Tags` | `TArray<FName>` | Filterable tags |
| `ThumbnailPath` | `FString` | Absolute path to the captured thumbnail image |
| `AudioClipRef` | `TSoftObjectPtr<USoundBase>` | Reference to the scratch audio asset |
| `LevelSequenceRef` | `TSoftObjectPtr<ULevelSequence>` | Reference to the shot's backing Level Sequence (populated after assemble) |
| `Steps` | `TArray<FPrevuStep>` | Ordered list of steps |

### FPrevuStep — Key Fields

| Field | Type | Description |
|-------|------|-------------|
| `Label` | `FString` | Step name (e.g. "A — Push in") |
| `ThumbnailPath` | `FString` | Path to the step's thumbnail |
| `SnapshotRef` | `TSoftObjectPtr<ULevelSnapshot>` | Reference to the Level Snapshot for this step |

## UPrevuProject

`UPrevuProject` is the root `UObject` asset for a Prevu production. It owns the act array and manages the sequences-out-of-date flag that drives the amber toolbar indicator.

### Staleness API

These three functions form the complete staleness contract. They are called automatically by Prevu's internal systems; you only need to call them directly if you are writing tooling that mutates the data model outside of Prevu's own UI.

```cpp
void MarkSequencesOutOfDate();
```

Sets the transient staleness flag. Call this after any mutation to the data model (adding/removing/reordering acts, scenes, shots, or steps; changing shot duration). The toolbar Assemble button will tint amber until the flag is cleared.

```cpp
void ClearSequencesOutOfDate();
```

Clears the staleness flag. Called automatically by `PrevuSequencerBridge::AssembleMasterSequence` on success. After this call the toolbar icon returns to its normal color.

```cpp
bool AreSequencesOutOfDate() const;
```

Returns `true` if the data model has been mutated since the last successful assemble. Use this to gate downstream operations that require up-to-date sequences.

### Acts Array

```cpp
TArray<FPrevuAct> Acts;
```

The top-level ordered list of acts. Mutate through Prevu's editor commands rather than directly, so that `MarkSequencesOutOfDate` is called and Slate views are notified.

## PrevuSequencerBridge

`PrevuSequencerBridge` lives in the `PrevuEditor` module. It is the only system that creates, modifies, or deletes `ULevelSequence` assets. All other Prevu systems are read-only with respect to Level Sequences.

### AssembleMasterSequence

```cpp
void AssembleMasterSequence(UPrevuProject* Project);
```

Rebuilds the full Master → Act → Scene → Shot Level Sequence hierarchy from the current state of `Project`. For each level of the hierarchy it creates the sub-sequence asset if it does not exist, nests it in its parent via a `UMovieSceneSubSection`, and sets the playback range from the shot's `DurationFrames` field. On success, calls `Project->ClearSequencesOutOfDate()`.

This is the **only** function that writes Level Sequence assets. Editing the data model through the UI never calls this function.

### DisplayToTick

```cpp
FFrameNumber DisplayToTick(int32 DisplayFrames, FFrameRate DisplayRate, FFrameRate TickResolution);
```

Converts a frame count expressed in display frames (e.g. 48 frames at 24 fps) to the corresponding `FFrameNumber` at the sequence's tick resolution (typically 24000 ticks per second). Applied at every `AddSequence` call inside `AssembleMasterSequence`.

!!! warning
    Always use `DisplayToTick` when passing frame positions to Sequencer sub-sequence APIs. Passing raw display-frame counts without conversion produces sequences that appear correct in the Sequencer timeline but are approximately 1000x shorter than intended due to the tick resolution mismatch.

## PrevuExporter

`PrevuExporter` lives in `PrevuEditor` and implements all three export formats.

### ExportToPNG

```cpp
bool ExportToPNG(const UPrevuProject* Project, const FString& OutputPath);
```

Composites all shot thumbnails, shot names, frame numbers, durations, and notes into a flat PNG image and writes it to `OutputPath`. Returns `true` on success.

### ExportToHTML

```cpp
bool ExportToHTML(const UPrevuProject* Project, const FString& OutputPath);
```

Writes a self-contained HTML storyboard sheet to `OutputPath`. All styles are inlined; the file has no external dependencies. Thumbnails are embedded as base64 data URIs. Returns `true` on success.

### ExportToBundle

```cpp
bool ExportToBundle(const UPrevuProject* Project, const FString& OutputPath);
```

Serializes the full `UPrevuProject` data model to a `.prevu` JSON file at `OutputPath`. The JSON captures all acts, scenes, shots, steps, metadata, notes, tags, and relative asset references. Binary assets (thumbnails, snapshots) are referenced by path, not embedded.

The corresponding import function in `PrevuImportExport` reads this file and reconstructs the `UPrevuProject` in memory.

## PrevuCapture

`PrevuCapture` lives in `PrevuEditor`. It manages SceneCapture2D-based thumbnail generation.

### CaptureShot

```cpp
bool CaptureShot(FPrevuShot& Shot, UWorld* World);
```

Places or reuses a `SceneCapture2D` actor in `World`, orients it to match the active camera, triggers a render, and writes the result to a path that is stored back on `Shot.ThumbnailPath`. Returns `true` if the capture succeeded.

### CaptureBatch

```cpp
void CaptureBatch(UPrevuProject* Project, UWorld* World);
```

Iterates every `FPrevuShot` in `Project` and calls `CaptureShot` for each. Progress is reported via the editor's task progress dialog. Shots that fail capture are skipped with a log warning; the batch continues.

## PrevuSnapshotManager

`PrevuSnapshotManager` lives in `PrevuEditor`. It wraps the engine's Level Snapshot system to associate snapshots with Prevu steps and apply `PrevuRestoreFilter` on restore.

### TakeSnapshot

```cpp
bool TakeSnapshot(FPrevuStep& Step, UWorld* World);
```

Captures the current state of `World` as a `ULevelSnapshot` asset and stores the asset reference on `Step.SnapshotRef`. Returns `true` on success.

### RestoreSnapshot

```cpp
bool RestoreSnapshot(const FPrevuStep& Step, UWorld* World);
```

Restores the Level Snapshot referenced by `Step.SnapshotRef` into `World`. The restore is filtered through `PrevuRestoreFilter`, which limits changes to only the actors and properties Prevu is responsible for. Returns `true` if the snapshot existed and the restore succeeded.

```cpp
bool HasSnapshot(const FPrevuStep& Step) const;
```

Returns `true` if `Step.SnapshotRef` is a valid, non-null reference.

## PrevuVocabularyDA

`PrevuVocabularyDA` is a `UDataAsset` subclass in the `Prevu` module. It centralizes the naming conventions and terminology used throughout the UI and exports. This allows a production to override default terms (for example, replacing "Shot" with "Setup" or "Step" with "Take") without recompiling.

| Property | Type | Description |
|----------|------|-------------|
| `ActLabel` | `FText` | Display name for the Act level (default: "Act") |
| `SceneLabel` | `FText` | Display name for the Scene level (default: "Scene") |
| `ShotLabel` | `FText` | Display name for the Shot level (default: "Shot") |
| `StepLabel` | `FText` | Display name for the Step level (default: "Step") |
| `DefaultShotDurationFrames` | `int32` | Duration assigned to new shots (default: 48) |
| `DefaultFrameRate` | `FFrameRate` | Project frame rate used for display and assemble (default: 24 fps) |

To use a custom vocabulary, create a `PrevuVocabularyDA` asset in the editor, configure it, and assign it in the Prevu project settings.

!!! note
    `PrevuVocabularyDA` affects display labels and export headers only. Internal type names (`FPrevuAct`, `FPrevuScene`, etc.) are fixed in C++.
