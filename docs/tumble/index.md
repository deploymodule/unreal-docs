# Tumble

Character ragdoll system — velocity-scaled physics drives, impulse API, and replication.

| | |
|---|---|
| **Category** | Gameplay |
| **UE Version** | 5.5+ |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **GAS Required** | No |
| **Replication** | Yes (server-authoritative) |

Tumble manages the transition between animated movement and physics ragdoll for characters. It uses velocity-scaled physical animation drives to blend smoothly between the current animation pose and full ragdoll, making death and stagger transitions feel weighted and physically believable. A simple impulse API lets you apply directional forces to individual bones (bullet impacts, explosions) without entering full ragdoll. Replication is handled automatically — clients receive authoritative ragdoll state.

## In This Section

- [Overview](overview.md) — Ragdoll architecture, drives, and blending
- [Usage](usage.md) — Setting up ragdoll and applying impulses
- [API Reference](api-reference.md) — Classes, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
