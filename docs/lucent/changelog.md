# Lucent — Changelog

## v1.0.0 — Initial Release

### Added
- Two-field DOF: near-field tiled scatter + far-field gather with physical Circle of Confusion
- Physical lens parameters: focal length, f-stop, focus distance, sensor width
- Custom bokeh shape support via `ULucentBokehAsset` (128×128 grayscale)
- Auto-focus target tracking with configurable interpolation speed
- Multi-scale physical bloom (6 scales, per-scale tint and intensity)
- Halation — lens-based red-channel highlight glow
- Anamorphic horizontal streak flares (1, 2, or 4 streak modes)
- Radial chromatic aberration with configurable radial falloff profile
- Brown-Conrady barrel/pincushion distortion (k1, k2, k3 coefficients)
- `ULucentSubsystem` for Blueprint and C++ runtime control
- Full CVar coverage under `r.Lucent.*` prefix
- Sequencer track (`ULucentSequencerTrack`) for animated focus pulls and iris changes
- LDR PIE compatibility via FloatRGBA UAV + blit pattern
- Half-resolution DOF passes with configurable `r.Lucent.DOF.ResScale`
- All passes independently togglable — disabled passes cost zero GPU time
