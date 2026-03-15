# Manifest — Changelog

## v1.0.0 — Initial Release

### Added
- `UManifestSubsystem` with `Resolve`, `ResolveSync`, `RegisterCatalog`, `UnregisterCatalog` APIs.
- `UManifestCatalog` Data Asset with `FManifestEntry` array (asset ID, display name, category, tags, soft path).
- `UManifestResolvedObject` runtime shell with load state, `GetObject`, and `GetAssetId`.
- `FManifestGuidPool` for deterministic, namespace-scoped GUID generation and validation.
- Catalog query APIs: `GetAssetsByCategory`, `GetAssetsByTag`, `IsRegistered`.
- Optional Courier integration for priority-aware loading (`bUseCourierForLoading` Project Setting).
- Hot-reload support: `RegisterCatalog` / `UnregisterCatalog` at runtime (DLC use case).
- `FOnManifestResolved` delegate for async resolution callbacks.
- Full Blueprint exposure for all subsystem functions.
