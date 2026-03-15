# Tumble — Changelog

## v1.0.0 — Initial Release

### Added
- `UTumbleComponent` — replicated ragdoll state machine (Animated / Blending / Ragdoll / GetUp)
- Velocity-scaled physical animation drives with per-body-region strength profiles
- `UTumbleBlendProfile` data asset for configuring drive strengths per body region
- `ApplyImpulse` API for hit reactions without entering full ragdoll
- Two impulse modes: Additive and Override
- Automatic get-up detection based on velocity stability threshold
- Face-up and face-down get-up animation montage support
- Capsule teleport to pelvis location on get-up sequence start
- Server-authoritative ragdoll physics with configurable client replication rate
- Delegates: `OnEnterRagdoll`, `OnExitRagdoll`, `OnGetUpStarted`
- Blueprint-callable API for all component functions
- Debug drive overlay via `r.Tumble.Debug 1`
- No GAS dependency
