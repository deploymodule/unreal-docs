# Aether

Anisotropic Kuwahara filter for painterly and watercolor post-processing effects.

| | |
|---|---|
| **Category** | Rendering |
| **UE Version** | 5.5+ (5.7 recommended) |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **Shader Model** | SM6 / DX12 required |
| **Hook** | `PrePostProcessPass_RenderThread` |

Aether applies the Anisotropic Kuwahara Filter (AKF) to the scene color buffer, transforming photorealistic renders into stylized oil-painting or watercolor imagery. The filter is orientation-aware — it aligns its smoothing kernel with the local structure tensor of the image, preserving edges while flattening textured surfaces into characteristic painterly strokes. Kernel size, orientation sensitivity, sharpness, and blend strength are all runtime-tunable, enabling smooth transitions between photorealistic and painterly looks.

## In This Section

- [Overview](overview.md) — Filter theory, structure tensor, and pipeline stages
- [Usage](usage.md) — Setup guide and blend examples
- [API Reference](api-reference.md) — CVars, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
