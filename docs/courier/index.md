# Courier

> Async asset loading with reference counting and priority.

Courier is a reference-counted async asset loading subsystem built on `FStreamableManager`. It tracks load states, supports per-request priority, and ensures assets stay loaded as long as any active requester holds a reference — then unloads them automatically.

| | |
|---|---|
| **Category** | Utility |
| **UE Version** | 5.5+ |
| **Blueprint** | Yes |
| **C++ API** | Yes |
| **Replication** | No |

## In This Section

- [Overview](overview.md)
- [Usage](usage.md)
- [API Reference](api-reference.md)
- [Changelog](changelog.md)
