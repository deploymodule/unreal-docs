# Gimbal — Changelog

## v1.0.0 — Initial Release

### Added
- `UGimbalComponent` with full priority-stack blending (linear, curve, instant).
- Six built-in camera modes: Third-Person, First-Person, Top-Down, Side-Scroller, Fixed, Orbit.
- `UGimbalModeAsset` Data Asset class for designer-friendly mode configuration.
- `UGimbalPresetAsset` for bundling named collections of modes.
- Lock-on targeting subsystem with candidate sorting by distance and screen centrality.
- `CycleLockOnTarget` API for next/previous target cycling.
- Six built-in `UGimbalBehavior` implementations: Offset, SpringArm, Collision, LockOn, FOV, Lag.
- Full Blueprint exposure for all runtime functions.
- Editor detail customization for `UGimbalModeAsset` with live preview support.
- CVars under `r.Gimbal.*` for runtime debug visualization.
