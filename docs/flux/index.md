# Flux

N-realm spatial zone-of-influence system for multi-dimensional or multi-state world spaces.

| | |
|---|---|
| **Category** | Gameplay |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Replication** | Yes |

Flux models a world that can exist in multiple "realms" simultaneously — each realm is a named layer with its own set of zone-of-influence volumes. Actors register with the Flux subsystem and are tracked against all active realm volumes. When an actor moves between zones or changes active realm, delegates fire to notify game systems. Common use cases include: elemental zones (fire/ice/void), faction territory, stealth detection radii, and biome influence blending.

## In This Section

- [Overview](overview.md) — Realm architecture and zone evaluation
- [Usage](usage.md) — Defining realms and zone volumes
- [API Reference](api-reference.md) — Classes, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
