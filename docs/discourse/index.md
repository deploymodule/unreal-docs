# Discourse

Data-driven conversation orchestrator — CSV-importable dialogue, staging system, and condition-gated lines.

| | |
|---|---|
| **Category** | Narrative |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Replication** | Optional (server-authoritative conversation state) |

Discourse manages structured conversations as Data Tables (importable from CSV) and orchestrates their playback through a staging system. Each conversation is a sequence of lines with optional speaker tags, conditions (querying Ledger facts), choices, and callbacks. The staging system handles one active conversation at a time per conversation manager, queuing subsequent conversations and firing delegates at each stage transition. No hardcoded UI — Discourse drives your UI through delegates.

## In This Section

- [Overview](overview.md) — Conversation data model and staging pipeline
- [Usage](usage.md) — Authoring conversations and triggering them
- [API Reference](api-reference.md) — Classes, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
