# Chronicle — Changelog

## v1.0.0 — Initial Release

### Added
- `UChronicleSubsystem` — `UWorldSubsystem` managing per-actor ring buffer transform recordings
- `UChronicleComponent` — auto-registering convenience component with configurable rate and duration
- Transform recording (location, rotation, scale) at configurable Hz
- Cubic Hermite interpolation for smooth position playback between frames
- Slerp interpolation for smooth rotation playback
- Four playback modes: Recording, Rewinding, Scrubbing, Replaying
- Configurable rewind speed multiplier
- Normalized scrub-to position (0–1 across the buffer)
- Replay from time offset with optional looping
- Optional skeletal per-bone recording for full character pose rewind
- Multi-actor management via single subsystem tick (no N component ticks)
- Delegates: `OnRewindStarted`, `OnRewindEnded`, `OnReplayStarted`, `OnReplayEnded`
- Recorded path debug visualization via `r.Chronicle.Debug 1`
- Blueprint-callable API for all subsystem and component functions
