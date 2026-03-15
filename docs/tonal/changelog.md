# Tonal — Changelog

## v1.0.0 — Initial Release

### Added
- Multi-stage HDR color grading compute pipeline (film stock → grain → halation → VHS → LUT)
- Four built-in film stock presets: Kodak Vision3 500T, Fuji Eterna 500, Kodak 2383 Print, Monochrome
- Custom curve authoring via `UTonalFilmCurveAsset` (per R/G/B channel toe, shoulder, gamma)
- Structured film grain with blue-noise temporal offset and luma-weighted distribution
- Halation simulation — red-channel separable Gaussian above configurable HDR threshold
- VHS mode with luma/chroma noise separation, scanline shimmer, and signal dropout
- 33-cube creative LUT import and blend support via `UTonalLUTAsset`
- `UTonalSubsystem` for Blueprint and C++ runtime control
- Full CVar coverage under `r.Tonal.*` prefix
- Post Process Volume integration via `ATonalPostProcessVolume`
- Sequencer track support (`UTonalSequencerTrack`) for animated grades
- LDR PIE compatibility via FloatRGBA UAV + blit pattern
- Editor detail panel with live preview for all parameters
