# Ledger

Genre-agnostic world state fact database — subsystem + gate component + Blueprint function library.

| | |
|---|---|
| **Category** | Gameplay |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Replication** | Server-authoritative |

Ledger provides a global key-value store for world state facts. Any system in the game can write facts (e.g., `"BossDefeated" = true`, `"PlayerGold" = 1200`), and any system can query them. `ULedgerGateComponent` blocks or allows actor interaction based on fact conditions — a door that only opens after the boss is dead, a dialogue node that only fires once. The Blueprint function library wraps the subsystem API for easy use in event graphs.

## In This Section

- [Overview](overview.md) — Architecture and fact types
- [Usage](usage.md) — Writing facts, querying, and gate conditions
- [API Reference](api-reference.md) — Classes, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
