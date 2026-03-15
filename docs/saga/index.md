# Saga

Data asset-driven quest and objective system for any genre.

| | |
|---|---|
| **Category** | Gameplay |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Replication** | Yes (subsystem authoritative) |

Saga manages quests, their objectives, and completion state through a simple subsystem API. Quests and tasks are defined entirely in Data Assets — no code changes are needed to add new quests. The system tracks active quests, objective progress, completion, and failure independently. Delegates fire at every state transition, making it easy to wire up UI, audio, and world events without coupling to the quest system internals.

## In This Section

- [Overview](overview.md) — Quest hierarchy, state machine, and architecture
- [Usage](usage.md) — Creating quests and tracking objectives
- [API Reference](api-reference.md) — Classes, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
