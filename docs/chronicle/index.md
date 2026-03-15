# Chronicle

Transform recording, rewind, and scrubbing system for actors and components.

| | |
|---|---|
| **Category** | Gameplay |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Replication** | Optional |

Chronicle continuously records actor transforms (location, rotation, scale) into a ring buffer and allows you to rewind, scrub, or replay that history at any time. Use it for bullet-time rewind mechanics, ghost replay features, time manipulation puzzles, or debugging. Recording rate, buffer duration, and interpolation quality are all configurable. Multiple actors can be recorded simultaneously by a single `UChronicleSubsystem` with minimal overhead.

## In This Section

- [Overview](overview.md) — Recording pipeline, ring buffer, and playback modes
- [Usage](usage.md) — Recording actors and triggering rewind
- [API Reference](api-reference.md) — Classes, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
