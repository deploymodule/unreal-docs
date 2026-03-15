# Marrow — Changelog

## v1.0.0 — Initial Release

### Added
- `UMarrowSanityComponent` — 0–100 sanity tracker with configurable drain and recovery rates
- Sanity threshold delegate system with configurable threshold values
- `OnSanityBroken` event at zero sanity
- `UMarrowDreadComponent` — short-term tension spike with exponential decay
- `UMarrowPerceptionComponent` — horror NPC multi-sense perception (hearing, vision, vibration, scent)
- Light-level sensitive vision range scaling
- Material-based audio occlusion for hearing
- Scent tag markers with world-space persistence and decay
- `UMarrowEnvironmentComponent` — composite `HorrorPressure` from darkness, isolation, temperature, sound
- Scene luma sampling for darkness measurement
- `UMarrowFlickerSubsystem` — batched light flicker management with named profiles
- `UMarrowFlickerProfile` data assets for configuring flicker frequency, variance, and fail patterns
- All systems independently usable
- Delegate-driven inter-system communication (no hardcoded coupling)
- Blueprint-callable API across all components
- Full replication for sanity and dread (server-authoritative)
