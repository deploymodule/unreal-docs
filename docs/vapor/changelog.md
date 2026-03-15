# Vapor — Changelog

## v1.0.0 — Initial Release

### Added
- Compute-shader-driven fog scattering pipeline (downsample → scatter → bilateral upsample)
- Rayleigh and Mie scattering with Henyey-Greenstein phase function
- Beer–Lambert height-based density falloff
- Up to 8 registered additional scatter lights (point and spot)
- Sky dome in-scatter contribution (`r.Vapor.SkyGlowIntensity`)
- Post Process Volume integration via `AVaporPostProcessVolume`
- Localized fog density with `AVaporFogVolume` actors
- `UVaporSubsystem` for Blueprint and C++ runtime control
- Full CVar coverage under `r.Vapor.*` prefix
- Permutation-based shader compilation (sky glow, additional lights, fog volumes as permutations)
- Editor detail panel with real-time CVar feedback
- Half-resolution scatter pass with configurable `r.Vapor.ResolutionScale`
- Edge-preserving bilateral upsample guided by full-resolution depth
- SM6 / DX12 validation on startup with user-friendly error message
- LDR PIE compatibility via FloatRGBA UAV + blit pattern
