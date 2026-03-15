# FastLoop — Changelog

## v1.0.0 — Initial Release

### Added
- `UFastLoopBFL` — Blueprint Function Library with 35+ native C++ array operations
- Filter operations: by class, tag, distance, float threshold, int equality, name substring
- Sort operations: actors by distance, floats ascending/descending, sort-by-index pattern
- Chunk operations: split actor and float arrays into fixed-size sub-arrays
- Single-chunk accessor (`GetChunk`) for frame-spread batch processing patterns
- Map/transform operations: actors to names, to locations, float scaling, element-wise addition
- Aggregate operations: sum, average, min, max, count, find-first, any, all
- `SelectByIndices` for combining sort-by-score with selection
- All functions Blueprint-callable as static nodes under **FastLoop** category
- Template implementations in C++ for typed usage without Blueprint overhead
- Zero dependencies — works without any other PluginDepot plugins
