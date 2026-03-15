# Sonance — Changelog

## v1.0.0 — Initial Release

### Added
- `USonanceSubsystem` with four managed sub-systems: Music, Ambience, Foley, Dialogue.
- `USonanceMusicTrack` Data Asset with per-stem MetaSound source references and roles.
- Stem weight API: `SetStemWeight` with fade duration for real-time music mixing.
- `USonanceAmbienceLayer` Data Asset with blend radius and intensity falloff curve.
- Push/Pop ambience layer stack with configurable blend-in/out durations.
- `SonanceFoleyBank` Data Asset with per-surface variation pools, pitch, and volume ranges.
- `USonanceFoleyComponent` for character footstep and impact foley routing.
- `USonanceDialogueComponent` with priority queue, interrupt support, and skip API.
- `USonanceDialogueData` with per-line locale-keyed audio map and subtitle text.
- Automatic locale selection from `FInternationalization` current culture.
- `OnDialogueComplete`, `OnLineBegin`, `OnLineComplete` delegates.
- Sound priority culling with graceful fade-out instead of hard stop.
- Full Blueprint exposure for all subsystem and component functions.
