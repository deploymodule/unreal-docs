# OmniSense — Changelog

## v1.0.0 — Initial Release

### Added
- `UOmniSenseComponent` with team ID, vision cone, acoustic sensitivity, and scent emission properties.
- `UOmniSenseSubsystem` with spatial hash grid and configurable frame-slice budget.
- Three sense types: Vision (frustum line-of-sight), Acoustic (sphere ping), Scent (token trail).
- `UOmniSenseTeamMatrix` Data Asset for team relationship configuration (Hostile/Neutral/Friendly).
- Occlusion result caching with configurable lifetime to reduce redundant traces.
- `OnStimulusDetected`, `OnStimulusLost`, `OnAcousticStimulus`, `OnScentStimulus` delegates.
- `FOmniStimulusEvent` struct with source actor, sense type, relationship, location, and confidence.
- Full Blueprint exposure for all component properties and subsystem functions.
- Debug drawing via `r.OmniSense.Debug` CVar (vision cones, acoustic rings, scent tokens).
