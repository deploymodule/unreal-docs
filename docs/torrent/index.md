# Torrent

Centralized Niagara effect lifecycle manager — pooling, budget control, and priority culling.

| | |
|---|---|
| **Category** | Performance |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Replication** | Server-spawned, client-simulated |

Torrent replaces ad-hoc `UNiagaraFunctionLibrary::SpawnSystem` calls with a managed pool that enforces a per-class budget, auto-culls low-priority effects when the budget is exceeded, and recycles expired systems back to the pool instead of destroying them. The result is predictable GPU particle cost with zero spikes from allocation. Effects register with a priority (Critical, High, Normal, Low) so the system always culls the least important effects first.

## In This Section

- [Overview](overview.md) — Pool architecture and budget system
- [Usage](usage.md) — Spawning, releasing, and configuring budgets
- [API Reference](api-reference.md) — Classes, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
