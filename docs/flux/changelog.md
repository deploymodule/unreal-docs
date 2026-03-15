# Flux — Changelog

## v1.0.0 — Initial Release

### Added
- `UFluxSubsystem` — `UWorldSubsystem` managing N realms and their zone-of-influence volumes
- `AFluxVolume` — actor defining realm spatial extent (box, sphere, or capsule shape)
- `UFluxComponent` — auto-registering actor component with cached influence values
- Smooth influence falloff at volume edges with configurable `FalloffRadius`
- Multi-volume additive influence within a single realm (capped at 1.0)
- `DominantRealm` tracking — the realm with the highest current influence
- Configurable query interval via `r.Flux.QueryInterval`
- Delegates: `OnDominantRealmChanged`, `OnInfluenceChanged`, `OnVolumeEntered`, `OnVolumeExited`
- Change threshold to prevent delegate spam on minor influence fluctuations
- Full Blueprint-callable API via subsystem and component
- Debug visualization via `r.Flux.Debug 1`
- Full replication of realm state
