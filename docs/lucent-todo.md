# Lucent — TODO

---

## 1. Implement Debug Visualization

The CVar `r.Lucent.Debug`, the panel dropdown, and the `ELucentDebugMode` enum are all wired up.
Nothing reads the CVar on the render thread and nothing in the shaders responds to it yet.

### Files to touch

| File | What needs to happen |
|---|---|
| `LucentViewExtension.h` — `FFrameData` | Add `int32 DebugMode = 0;` |
| `LucentViewExtension.h` — `FLucentLDRParameters` | Add `SHADER_PARAMETER(int32, DebugMode)` |
| `LucentViewExtension.h` — `FLucentHDRParameters` | Add `SHADER_PARAMETER(int32, DebugMode)` (needed for HDR buffer visualizations) |
| `LucentViewExtension.cpp` — `BeginRenderViewFamily` | Read `CVarLucentDebug` into `NewData.DebugMode`, set `HDRParams.DebugMode` and `LDRParams.DebugMode` |
| `LucentViewExtension.cpp` — pass a reference to intermediate buffers into `RenderHDRComposite` / `RenderLDRStage` | HDR-side debug modes need access to `HighlightBuffer`, `GlowBuffer`, `HalationBuffer`, `StreaksBuffer` |
| `LucentHDRComposite.usf` | Override `Result` based on `LucentHDR.DebugMode` (modes 1–5) |
| `LucentLDRComposite.usf` | Override `Color` based on `LucentLDR.DebugMode` (modes 6–8) |

### Debug mode assignments (matches `ELucentDebugMode`)

| Value | Name | Where | What to output |
|---|---|---|---|
| 0 | Off | — | Normal render |
| 1 | Highlight Mask | HDR composite | `HighlightBuffer` sample (pre-bloom threshold mask) |
| 2 | Bloom Buffer | HDR composite | `GlowBuffer` sample (bloom accumulation before composite) |
| 3 | Halation Buffer | HDR composite | `HalationBuffer` sample |
| 4 | Streak Buffer | HDR composite | `StreaksBuffer` sample |
| 5 | Dirt-Modulated Glow | HDR composite | `GlowBuffer * DirtMask` |
| 6 | Distortion Grid | LDR composite | Visualize distortion warp — draw a grid in logical UV space and show it distorted |
| 7 | Vignette Mask | LDR composite | Output grayscale vignette weight (what `EllipticalVignette` returns) |
| 8 | Grain Only | LDR composite | Output `Noise * ResponseMask` as grayscale |

### Notes
- HDR modes (1–5): `RenderHDRComposite` already receives all the buffers as parameters — just pass `DebugMode` in the uniform and branch in the shader.
- LDR modes (6–8): already computed inside `LucentLDRComposite.usf` — just add a branch at the end before `OutputTexture[PixelPos]`.
- Modes 1–5 should short-circuit `FinalGlow` and replace `Result = <debug buffer>` instead of `Scene + FinalGlow`.
- For mode 6 (Distortion Grid), a simple approach: generate a checkerboard/grid in logical UV space, then sample it at `DistortedUV` to show the warp shape.

---

## 2. Remove Async Compute Entirely

Async compute on a serial dependent downsample/upsample chain adds queue-transition overhead and cache pressure without any parallelism benefit. Remove the feature completely.

### Removal checklist

- [ ] **`LucentCVars.cpp`** — delete the `CVarLucentAsyncCompute` definition
- [ ] **`LucentCVars.h`** — delete the `extern TAutoConsoleVariable<int32> CVarLucentAsyncCompute;` declaration
- [ ] **`LucentViewExtension.h`** — `FFrameData`: delete `bool bAsyncCompute = false;`
- [ ] **`LucentViewExtension.cpp`** — `BeginRenderViewFamily`: delete the line reading `CVarLucentAsyncCompute`
- [ ] **`LucentViewExtension.cpp`** — `RenderDownsampleChain`: remove `BlurPassFlags` conditional, hardcode `ERDGPassFlags::Compute`
- [ ] **`LucentViewExtension.cpp`** — `RenderUpsampleChain`: same as above
- [ ] **`LucentSettings.h`** — delete `bAsyncCompute` property (if present)
- [ ] **`LucentSettings.cpp`** — delete `bAsyncCompute` from `ApplyToCVars`, `ReadFromCVars`, `Lerp`
- [ ] **`LucentPanel.cpp`** — delete the `MakeBoolRow` for `r.Lucent.AsyncCompute` in the Performance section
- [ ] **`LucentPanel.cpp`** — if the Performance section is now empty (only Preserve Alpha remains), keep the section; just verify it still looks right
