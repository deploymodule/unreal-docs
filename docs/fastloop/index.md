# FastLoop

High-performance Blueprint array-processing library — filter, sort, chunk, map, and reduce.

| | |
|---|---|
| **Category** | Performance |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |

FastLoop provides functional array processing nodes for Blueprint that are significantly faster than equivalent Blueprint loop constructs. All operations are implemented in native C++ and exposed as Blueprint-callable static functions. Because Blueprint `ForEachLoop` nodes have per-iteration overhead from the VM, FastLoop's native equivalents can be 10–50x faster on large arrays (1000+ elements). Particularly useful for AI scoring arrays, inventory sorting, and data processing in game systems.

## In This Section

- [Overview](overview.md) — Operation categories and performance notes
- [Usage](usage.md) — Examples for each operation type
- [API Reference](api-reference.md) — Full function list
- [Changelog](changelog.md) — Version history
