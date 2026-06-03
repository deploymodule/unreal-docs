# Prevu — Changelog

## v1.0.0 — Initial Release

UE 5.7 · Win64 · Zachanot LLC

### Added

**Core data model**

- `UPrevuProject` root asset with a four-level hierarchy: `TArray<FPrevuAct>` → `TArray<FPrevuScene>` → `TArray<FPrevuShot>` → `TArray<FPrevuStep>`
- `FPrevuAct` struct with name and ordered scene array
- `FPrevuScene` struct with name and ordered shot array
- `FPrevuShot` struct with duration (frames), notes, tags, thumbnail path, audio clip reference, Level Sequence reference, and ordered step array
- `FPrevuStep` struct with label, thumbnail path, and Level Snapshot asset reference
- `UPrevuProject::MarkSequencesOutOfDate` / `ClearSequencesOutOfDate` / `AreSequencesOutOfDate` transient staleness API, set on every data model mutation and cleared on successful assemble
- `PrevuShotTemplate` asset type for pre-configured shot defaults
- `PrevuVocabularyDA` data asset for configurable display terminology (Act/Scene/Shot/Step labels, default duration, default frame rate)
- Blueprint function library exposing key read operations to Blueprint graphs

**Slate UI (PrevuEditor module)**

- `SPrevuTab` — main dockable editor tab, hosts all sub-panels
- `SPrevuHierarchy` — Act/Scene/Shot/Step tree with expand/collapse, rename (double-click), drag-to-reorder, and contextual add/remove
- `SPrevuBoardView` — thumbnail grid storyboard view, synchronized selection with the hierarchy panel
- `SPrevuShotInspector` — right-side inspector showing duration, notes, tags, steps list with per-step snapshot status, audio clip reference, and Level Sequence reference
- `SPrevuToolbar` — toolbar with Assemble button (amber tint + re-assemble tooltip when sequences are stale), New Act, New Scene, New Shot, Export, and Import buttons
- Viewport HUD with letterbox overlay, shot label, and take counter

**Sequencer assembly**

- `PrevuSequencerBridge::AssembleMasterSequence` — builds the full Master → Act → Scene → Shot `ULevelSequence` nesting from the current data model; the sole path that creates or modifies Level Sequence assets
- `DisplayToTick` helper that converts display-frame counts to tick-resolution `FFrameNumber` values at the correct ratio, applied at all `AddSequence` sites to prevent the ~1000x duration mismatch caused by passing raw display frames to Sequencer sub-sequence APIs
- Assemble clears the staleness flag on success; any data model mutation after assemble immediately re-sets it

**Thumbnail capture**

- `PrevuCapture::CaptureShot` — places a `SceneCapture2D` actor, orients it to the active camera, triggers a render, and writes the result to the shot's thumbnail path
- `PrevuCapture::CaptureBatch` — iterates every shot in the project with editor task progress reporting; failed captures are skipped with a log warning

**Level Snapshots**

- `PrevuSnapshotManager::TakeSnapshot` — captures current level state as a `ULevelSnapshot` and stores the reference on a `FPrevuStep`
- `PrevuSnapshotManager::RestoreSnapshot` — restores a step's snapshot with `PrevuRestoreFilter` to limit actor/property scope
- `PrevuRestoreFilter` — selective restore filter restricting snapshot application to Prevu-managed actors and properties, preventing unintended side-effects on unrelated level content
- `PrevuSnapshotManager::HasSnapshot` — null-safe check for whether a step has a valid snapshot reference

**Scratch audio**

- `PrevuAudioManager` — scratch audio recording routed through the engine's Sequencer Take Recorder integration; audio clip references stored on `FPrevuShot`
- Integration with the engine **Takes** plugin (`TakesCore` + `TakeRecorder`)

**Take management**

- `PrevuTakeManager` — take numbering and management per shot; take counter reflected in the viewport HUD

**Export and import**

- `PrevuExporter::ExportToPNG` — composites shot thumbnails, names, durations, and notes into a flat PNG storyboard sheet
- `PrevuExporter::ExportToHTML` — produces a self-contained HTML storyboard sheet with base64-embedded thumbnails and inlined styles; no external dependencies
- `PrevuExporter::ExportToBundle` — serializes the full `UPrevuProject` to a `.prevu` JSON file including all metadata, notes, tags, and relative asset references
- `PrevuImportExport` — re-imports a `.prevu` JSON bundle and reconstructs the `UPrevuProject` data model
- Export dialog accessible from the toolbar Export button

**Plugin packaging**

- `Prevu` module declared `UncookedOnly` (not compiled into packaged games); `PrevuEditor` module declared `Editor`
- `Prevu.uplugin` with `VersionName 1.0.0`, `EngineVersion` targeting UE 5.7, `Win64` target platform, `CreatedBy "Zachanot LLC"`, and explicit dependency on the engine **Takes** plugin
- `Resources/Icon128.png` placeholder icon
- Copyright headers (`// Copyright 2026 Zachanot LLC. All Rights Reserved.`) on all 79 source files
- Proprietary LICENSE file included in the plugin root
