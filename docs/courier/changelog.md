# Courier — Changelog

## v1.0.0 — Initial Release

### Added
- `UCourierSubsystem` wrapping `FStreamableManager` with reference counting and priority queuing.
- `FCourierHandle` RAII handle — asset stays loaded while any handle is alive, unloads automatically on last release.
- `FCourierBatchHandle` for multi-asset batch loads with `GetProgress` and `IsComplete` queries.
- Four priority levels: Low, Normal, High, Critical.
- `ECourierLoadState` enum: Unloaded, Loading, Loaded, Failed.
- `PreloadByTag` integration with `UAssetManager` for GameplayTag-driven preloading.
- `CancelLoad` to abort pending in-flight requests.
- `FOnCourierLoaded`, `FOnCourierBatchLoaded`, `FOnCourierLoadFailed` delegates.
- Full Blueprint exposure for all subsystem functions.
- Debug logging via `r.Courier.Verbose` CVar showing reference counts and load states.
