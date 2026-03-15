# Tonal Plugin Expansion — Implementation Tracker

## Status Key
- `[ ]` Not started
- `[~]` In progress
- `[x]` Complete

---

## Step 1 — Black Floor Compression (HDR)
_Small additive change to existing HDR shader._

- [x] `TonalCVars.h/.cpp` — Add `r.Tonal.Shaper.BlackFloor` CVar (float, 0.0)
- [x] `TonalViewExtension.cpp` — Add `BlackFloor` to `FTonalHDRParameters` struct and fill from `FFrameData`
- [x] `TonalViewExtension.h` — Add `float BlackFloor` field to `FFrameData`
- [x] `TonalHDRShaper.usf` — Apply floor in `ApplyHDRShaper` (cubic rolloff, max +3% lift at pure black)
- [x] `TonalSettings.h/.cpp` — Add `ShaperBlackFloor` UPROPERTY + sync
- [x] `TonalPanel.cpp` — Add slider under HDR Shaper section

---

## Step 2 — LDR Color Science Group
_Three features added to `FTonalLDRParameters`. No new permutations — passthrough at Strength=0._

### 2a. Highlight Saturation Compression
- [x] `TonalCVars.h/.cpp` — `r.Tonal.HighlightDesat.Enabled`, `.Threshold`, `.Strength`
- [x] `TonalViewExtension.cpp` — Add params to `FTonalLDRParameters` + `FFrameData`
- [x] `TonalLDRFusion.usf` — Implement desaturation above luma threshold
- [x] `TonalSettings.h/.cpp` — Add fields + sync
- [x] `TonalPanel.cpp` — New "Color Science" section

### 2b. Midtone Saturation Bias
- [x] `TonalCVars.h/.cpp` — `r.Tonal.MidtoneSat.Enabled`, `.Strength`, `.Center`, `.Range`
- [x] `TonalViewExtension.cpp` — Add params
- [x] `TonalLDRFusion.usf` — Gaussian bell-curve sat boost in mid-luma range
- [x] `TonalSettings.h/.cpp` — Add fields + sync
- [x] `TonalPanel.cpp` — Sliders in "Color Science" section

### 2c. Highlight Hue Shift
- [x] `TonalCVars.h/.cpp` — `r.Tonal.HueShift.Enabled`, `.Strength`, `.Threshold`
- [x] `TonalViewExtension.cpp` — Add params
- [x] `TonalLDRFusion.usf` — Warm RGB shift above luma threshold (+R, -B)
- [x] `TonalSettings.h/.cpp` — Add fields + sync
- [x] `TonalPanel.cpp` — Sliders in "Color Science" section

---

## Step 3 — HDR Midtone Glow
_New RDG pass after the shaper. Multi-tap Kawase blur masked to mid-luma range (0.2–0.6), additive blend._

- [x] `TonalCVars.h/.cpp` — `r.Tonal.MidtoneGlow.Enabled`, `.Intensity`, `.Radius`, `.LumaMin`, `.LumaMax`
- [x] `TonalViewExtension.h` — Add midtone glow fields to `FFrameData` + `RenderHDRMidtoneGlow()` declaration
- [x] `TonalViewExtension.cpp` — `FTonalMidtoneGlowParameters` UB + `FTonalMidtoneGlowCS` + `RenderHDRMidtoneGlow()` impl chained after shaper
- [x] `Shaders/TonalMidtoneGlow.usf` — 8-tap Kawase ring, quadratic mid-luma mask, additive blend
- [x] `TonalSettings.h/.cpp` — Add fields + sync + Lerp
- [x] `TonalPanel.cpp` — "Midtone Glow (HDR)" section between HDR Shaper and Micro-Contrast

---

## Step 4 — LDR Skin Softener
_New permutation `TONAL_SKIN_SOFTENER_ENABLED`. Multi-tap selective blur, skin-tone hue/luma mask. New debug mode 4._

- [x] `TonalCVars.h/.cpp` — `r.Tonal.SkinSoftener.Enabled`, `.Strength`, `.LumaMin`, `.LumaMax`, `.HueCenter`, `.HueRange`
- [x] `TonalViewExtension.h` — Add skin softener fields to `FFrameData`
- [x] `TonalViewExtension.cpp` — Add params to `FTonalLDRParameters`; add `FSkinSoftenerEnabledDim` permutation
- [x] `TonalLDRFusion.usf` — Skin-tone detection (approx hue from RGB) + selective softening + `DebugMode == 4` mask output
- [x] `TonalCVars.cpp` — Extend `r.Tonal.DebugView` tooltip to include mode 4
- [x] `TonalSettings.h/.cpp` — Add fields + sync
- [x] `TonalPanel.cpp` — New "Skin Softener" section; add mode 4 to debug dropdown

---

## Step 5 — LDR Depth Fade
_New permutation `TONAL_DEPTH_FADE_ENABLED`. Requires SceneDepth SRV + camera linearization params in LDR stage._

- [x] `TonalCVars.h/.cpp` — `r.Tonal.DepthFade.Enabled`, `.NearDistance`, `.FarDistance`, `.ContrastFalloff`, `.SatFalloff`
- [x] `TonalViewExtension.h` — Add depth fade fields + camera params to `FFrameData`
- [x] `TonalViewExtension.cpp` — Bind `SceneDepth` SRV + projection params (`InvDeviceZToWorldZTransform`) to LDR pass; add `FDepthFadeEnabledDim` permutation
- [x] `TonalLDRFusion.usf` — Linearize depth, compute fade mask, apply contrast + sat falloff
- [x] `TonalSettings.h/.cpp` — Add fields + sync
- [x] `TonalPanel.cpp` — New "Depth Fade" section

---

## Notes

### Permutation count after all steps
| Dim | Added in step |
|---|---|
| `TONAL_MICRO_CONTRAST_ENABLED` | Original |
| `TONAL_SPLIT_TONE_ENABLED` | Original |
| `TONAL_SUBJECT_EMPHASIS_ENABLED` | Original |
| `TONAL_SKIN_SOFTENER_ENABLED` | Step 4 |
| `TONAL_DEPTH_FADE_ENABLED` | Step 5 |

Total: 2^5 = **32 permutations** — Color Science features use no permutation (Strength=0 passthrough).

### Spec binding note
The spec says to avoid manual HLSL declarations. This is incorrect for UE5 RDG — per-dispatch resources (Texture2D, SamplerState, RWTexture2D) and SHADER_PARAMETER scalars require manual declaration in USF. Only `BEGIN_GLOBAL_SHADER_PARAMETER_STRUCT` members are auto-injected. Current architecture is correct.
