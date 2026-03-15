# Cadence

Input-sequence combo matcher — pure pattern detection, delegate-driven, no GAS dependency.

| | |
|---|---|
| **Category** | Gameplay |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Replication** | Client-side |

Cadence solves a specific problem cleanly: detecting ordered input sequences (combos) and firing delegates when they match. It is entirely decoupled from animation, ability execution, and game logic — it only detects patterns and reports them. What happens when a combo fires is entirely up to the consumer. Define combo sequences as Data Assets, register them with the `UCadenceComponent`, and bind to `OnComboMatched`.

## In This Section

- [Overview](overview.md) — Pattern matching architecture and combo evaluation
- [Usage](usage.md) — Defining combos and binding to events
- [API Reference](api-reference.md) — Classes, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
