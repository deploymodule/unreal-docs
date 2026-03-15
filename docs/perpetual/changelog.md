# Perpetual — Changelog

## v1.0.0 — Initial Release

### Added
- `UPerpetualAnimationComponent` with full stage machine: `GoToStage`, `Advance`, `Reverse`, `Pause`, `Resume`.
- `UPerpetualAnimationAsset` Data Asset with multi-stage sequences, auto-chain, reverse, loop, and delay support.
- `FPerpetualStage` struct with per-stage transform, easing curve, audio cues, and next/reverse stage references.
- Replication via `FPerpetualStageState` — server authoritative, clients interpolate locally.
- `OnStageBegin`, `OnStageComplete`, and `OnSequenceComplete` delegates exposed to Blueprint and C++.
- Multi-component support: drive multiple `USceneComponent` targets from a single asset.
- Looping stage support for fans, rotating platforms, oscillating props.
- Full Blueprint exposure for all trigger and query functions.
