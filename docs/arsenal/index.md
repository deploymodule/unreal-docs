# Arsenal

Equipment and weapon system — GAS-independent, driven entirely by hard-reference Data Assets.

| | |
|---|---|
| **Category** | Gameplay |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Replication** | Yes |

Arsenal is a self-contained equipment and weapon management system. Items are defined as Data Assets (`UArsenalItemData`) — no soft references, no async loading, no GAS dependency. The component (`UArsenalComponent`) handles equip/unequip logic, slot management, stat aggregation, and visual attachment. Because everything is hard-referenced, items are always in memory and ready with zero latency, which is ideal for fast-paced action games.

## In This Section

- [Overview](overview.md) — Architecture, slots, and stat flow
- [Usage](usage.md) — Creating items, equipping, and querying stats
- [API Reference](api-reference.md) — Classes, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
