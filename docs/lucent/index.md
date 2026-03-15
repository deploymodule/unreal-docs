# Lucent

Cinematic lens stack — depth of field, bloom, halation, anamorphic streaks, chromatic aberration, barrel distortion, and VFX overlays.

| | |
|---|---|
| **Category** | Rendering |
| **UE Version** | 5.5+ (5.7 recommended) |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **Shader Model** | SM6 / DX12 required |
| **Hook** | `PrePostProcessPass_RenderThread` |

Lucent models the full optical path of a cinema prime lens as a series of compute shader passes. Each element — from the bokeh-shaped depth of field to physically-sized anamorphic streak flares — is individually togglable so you pay only for what you use. An integrated Sequencer track lets you animate focus pulls, iris changes, and aberration builds across a cut.

## In This Section

- [Overview](overview.md) — Lens stack architecture and each optical element
- [Usage](usage.md) — Setup guide with per-element examples
- [API Reference](api-reference.md) — CVars, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
