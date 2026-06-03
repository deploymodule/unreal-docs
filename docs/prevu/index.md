# Prevu

A storyboard and animatic authoring system for Unreal Engine, built in C++ and Slate.

Prevu organizes a production into a four-level hierarchy — Act, Scene, Shot, and Step — and backs every shot with a real `ULevelSequence`. From inside the editor you capture thumbnails, record scratch audio, take Level Snapshots per step, and assemble a nested master sequence without leaving Unreal. When you are ready to share or archive, Prevu bundles everything into a portable `.prevu` JSON package.

- Four-level hierarchy: Act, Scene, Shot, Step
- Every shot backed by a real `ULevelSequence`; nested master sequence assembled on demand
- SceneCapture2D thumbnail capture per shot, with batch mode
- Level Snapshot saved per step with selective restore via `PrevuRestoreFilter`
- Scratch audio recording routed through Sequencer
- Storyboard sheet export as PNG and as a self-contained HTML file
- Portable `.prevu` JSON bundle for version control and sharing
- Amber toolbar indicator signals when assembled sequences are stale

UE 5.7 · Editor only · Zachanot LLC

---

- [Overview](overview.md) — architecture, hierarchy model, sync model, export formats
- [Usage](usage.md) — step-by-step workflows for every major feature
- [API Reference](api-reference.md) — data model types, key functions, bridge and exporter APIs
- [Changelog](changelog.md) — release history
