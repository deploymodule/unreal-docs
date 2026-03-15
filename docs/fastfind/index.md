# FastFind

High-performance Blueprint spatial query library — broadphase sphere queries with actor filtering.

| | |
|---|---|
| **Category** | Performance |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Thread-Safe** | Yes (read-only queries) |

FastFind provides optimized spatial query functions as Blueprint-callable static nodes. Standard Blueprint solutions for "find actors within radius" use `GetAllActorsOfClass` (iterates every actor in the world) or `OverlapSphereForObjects` (full physics overlap). FastFind uses a custom broadphase — a flat hash grid updated incrementally — to return results in O(1) average time regardless of world size. Designed for AI systems, AOE abilities, and detection systems that need to query frequently every frame.

## In This Section

- [Overview](overview.md) — Broadphase grid architecture and query types
- [Usage](usage.md) — Registering actors and running queries
- [API Reference](api-reference.md) — Functions and Blueprint nodes
- [Changelog](changelog.md) — Version history
