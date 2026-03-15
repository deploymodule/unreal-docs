# Optica — Changelog

## v1.0.0 — Initial Release

### Added
- `UOpticaComponent` — unified weapon feel and camera motion component
- Weapon sway: lag, tilt, and vertical channels with per-channel spring parameters
- `UOpticaSwayProfile` data asset for configurable sway per weapon type
- Spring-based dual recoil system: independent camera spring and weapon spring
- `UOpticaRecoilProfile` data asset for per-weapon recoil characteristics
- ADS transition with FOV blend, sway scale reduction, and weapon socket lerp
- `HoldBreath` API for reduced sway during sustained ADS
- Procedural breathing layer with exertion-based rate scaling
- Footstep camera bob synced to movement speed
- Sensor emulation render pass: grain, exposure lag, motion blur scaling, vignette pulse
- `UOpticaViewExtension` running sensor emulation at `PrePostProcessPass_RenderThread`
- Full CVar coverage under `r.Optica.Sensor.*`
- Blueprint-callable API for all component functions
- Replication of ADS state and recoil events for networked play
