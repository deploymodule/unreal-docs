# Lean — Changelog

## v1.0.0 — Initial Release

### Added
- `ULeanComponent` with full 360-degree lean via 2D direction vector.
- Collision-aware lean clamping via configurable sphere trace (radius, length, channel).
- Smooth interpolation with configurable lean speed.
- `SetLeanInput` (1D) and `SetLeanDirection` (2D) input APIs.
- Optional Enhanced Input action binding for zero-boilerplate integration.
- Replication via compressed `int8` lean value; proxies interpolate locally.
- `OnLeanObstructed` and `OnLeanCleared` delegates.
- `GetObstructedFraction` for Blueprint-driven cover awareness feedback (e.g., HUD indicator).
- Full Blueprint exposure for all component properties and functions.
