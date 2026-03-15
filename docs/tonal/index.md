# Tonal

HDR/LDR color grading with film stock emulation, halation, and VHS analog artifacts.

| | |
|---|---|
| **Category** | Rendering |
| **UE Version** | 5.5+ (5.7 recommended) |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **Shader Model** | SM6 / DX12 required |
| **Hook** | `PrePostProcessPass_RenderThread` |

Tonal provides a complete color grading pipeline modeled after real photochemical film stocks. It operates in HDR scene linear space before tone mapping, ensuring physically accurate emulsion response curves, grain structure, and halation glows. A VHS mode adds analog tape artifacts including luma/chroma separation noise, scanline flicker, and signal dropout for retro aesthetics.

The pipeline is structured as independent stages that can be toggled individually, so you pay only for the effects you enable.

## In This Section

- [Overview](overview.md) — Pipeline stages and architecture
- [Usage](usage.md) — Setup guide with film stock presets
- [API Reference](api-reference.md) — CVars, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
