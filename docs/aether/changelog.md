# Aether — Changelog

## v1.0.0 — Initial Release

### Added
- Anisotropic Kuwahara Filter (AKF) post-process pipeline (structure tensor → smooth → AKF → composite)
- Generalized 8-sector anisotropic filter aligned to local structure tensor eigenvectors
- 4-sector performance mode via `r.Aether.SectorCount 4`
- Structure tensor Gaussian smoothing with configurable radius
- Blend strength parameter for partial stylization (0 = original scene, 1 = full painterly)
- Configurable kernel radius, sharpness exponent, and orientation sensitivity
- Half-resolution render option via `r.Aether.ResScale`
- `UAetherSubsystem` for Blueprint and C++ runtime control
- Full CVar coverage under `r.Aether.*` prefix
- Post Process Volume integration via `AAetherPostProcessVolume`
- Real-time blend animation support for stylization transition effects
- Editor detail panel with live preview for all filter parameters
- LDR PIE compatibility via FloatRGBA UAV + blit pattern
