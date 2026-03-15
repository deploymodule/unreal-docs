# FastFind — Changelog

## v1.0.0 — Initial Release

### Added
- `UFastFindSubsystem` — `UWorldSubsystem` managing a flat hash grid of registered actors
- Flat hash grid with configurable cell size (`r.FastFind.CellSize`)
- Lazy grid update at configurable rate (`r.FastFind.UpdateRate`)
- Static actor optimization — static actors never re-inserted after initial placement
- Six query types: radius, radius-by-class, radius-by-tag, nearest, N nearest, cone
- Template `FindActorsInRadiusByClass<T>` for typed C++ queries
- `UFastFindComponent` — auto-registering convenience component
- Update priority levels: Static, Dynamic, Frequent
- `UFastFindBFL` — Blueprint function library with static nodes for all queries
- Debug grid cell visualization via `r.FastFind.Debug 1`
- O(1) average query time regardless of total actor count
- No physics dependency — runs entirely on game thread
