# Vapor

Enhanced volumetric fog scattering driven by a compute-shader downsample/upsample pipeline.

| | |
|---|---|
| **Category** | Rendering |
| **UE Version** | 5.5+ (5.7 recommended) |
| **Blueprint API** | Yes |
| **C++ API** | Yes |
| **Shader Model** | SM6 / DX12 required |
| **Hook** | `PrePostProcessPass_RenderThread` |

Vapor replaces Unreal's default exponential height fog with a physically-motivated light-scattering pipeline that accounts for directional lights, additional point/spot lights, and sky dome contribution. The full effect runs in a half-resolution compute pass then upsamples back to native resolution with edge-preserving bilateral filtering.

Parameters are exposed as CVars (for instant GPU feedback), as a settings `UObject` (for Blueprints and the detail panel), and via Post Process Volume integration so you can blend fog zones spatially.

## In This Section

- [Overview](overview.md) — Architecture and how the pipeline works
- [Usage](usage.md) — Step-by-step setup guide with examples
- [API Reference](api-reference.md) — CVars, functions, and Blueprint nodes
- [Changelog](changelog.md) — Version history
