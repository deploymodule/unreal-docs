# Tonal Features v2 — Implementation Todo

## Status Key
- `[ ]` Not started
- `[~]` In progress
- `[x]` Complete

---

## Architecture Reference

### LDR Fusion Execution Order (post-implementation)

```
1.  Skin Softener           [existing, permutation]
2.  Micro-Contrast          [existing, permutation]
3.  Split Tone              [existing, permutation]
4.  Bleach Bypass           [NEW — Feature C, inline strength check]
5.  Color Science group     [existing + new, inline strength checks]
    5a. Highlight Desaturation
    5b. Midtone Saturation Bias
    5c. Highlight Hue Shift
    5d. Hue-Select Slots    [NEW — Feature A, inline slot-count check]
6.  Subject Emphasis        [existing, permutation]
7.  Depth Fade              [existing, permutation]
8.  Color Temp by Depth     [NEW — Feature D, permutation]
9.  Light Wrap              [NEW — Feature F, permutation]
→   SanitizeColor + Output
```

### HDR Pipeline Order (post-implementation)

```
Pass 1: TonalHDRShaper           [existing]
Pass 2: TonalFilmStock           [NEW — Feature B, conditionally dispatched]
Pass 3: TonalMidtoneGlow         [existing]
```

### LDR Pipeline Order (post-implementation)

```
Pre-pass: TonalLocalToneBase         [NEW — Feature E, conditionally dispatched]
Pre-pass: TonalLocalToneReconstruct  [NEW — Feature E, conditionally dispatched]
Main:     TonalLDRFusion             [existing + all additions]
```

### Updated Permutation Table

| Dimension | Source |
|---|---|
| `TONAL_MICRO_CONTRAST_ENABLED` | Original |
| `TONAL_SPLIT_TONE_ENABLED` | Original |
| `TONAL_SUBJECT_EMPHASIS_ENABLED` | Original |
| `TONAL_SKIN_SOFTENER_ENABLED` | v1 Step 4 |
| `TONAL_DEPTH_FADE_ENABLED` | v1 Step 5 |
| `TONAL_DEPTH_WB_ENABLED` | Feature D (v2) |
| `TONAL_LIGHT_WRAP_ENABLED` | Feature F (v2) |

**Total: 2^7 = 128 LDR fusion permutations.**

Bleach Bypass and Hue-Select Slots have no permutations — pure ALU,
inline strength/count checks. Film Stock and Local Tone Mapping are
conditionally dispatched full passes (zero cost when disabled).

### Debug Mode Registry

| Mode | Shader | Visualizes |
|---|---|---|
| 1 | `TonalHDRShaper.usf` | Curve delta heatmap |
| 2 | `TonalLDRFusion.usf` | Micro-contrast mask |
| 3 | `TonalLDRFusion.usf` | Subject emphasis mask |
| 4 | `TonalLDRFusion.usf` | Skin detection mask |
| 5 | `TonalLDRFusion.usf` | Depth fade mask |
| 6 | `TonalLDRFusion.usf` | **NEW** Hue-select composite mask (all active slots combined) |
| 7 | `TonalLDRFusion.usf` | **NEW** Depth WB zones (green=near, yellow=mid, red=far) |
| 8 | `TonalLDRFusion.usf` | **NEW** Light wrap edge mask |
| 1 | `TonalFilmStock.usf` | **NEW** Per-channel delta heatmap (own CVar) |
| 1 | `TonalLocalToneReconstruct.usf` | **NEW** Base layer only (own CVar) |
| 2 | `TonalLocalToneReconstruct.usf` | **NEW** Detail layer only (own CVar) |

### Depth Binding Rule
`SceneDepth` SRV must be bound whenever **any** depth-reading feature is active.
Controlled by `bool bNeedsDepth` in `TonalViewExtension.cpp`:

```cpp
bool bNeedsDepth = FrameData.bDepthFadeEnabled
                || FrameData.bDepthWBEnabled
                || (FrameData.bLightWrapEnabled && FrameData.bLightWrapUseDepth);
```

**Never** gate depth binding on a single permutation. See Bug Fix §1.

---

## Bug Fixes

These are correctness issues — silent failures or breaking artifacts — that must be
implemented before or alongside the new features.

---

### BUG-1 — Depth SRV bound only when Depth Fade is active

**Root cause:** The C++ binding for `SceneDepth` is conditional on `TONAL_DEPTH_FADE_ENABLED`.
Color Temp by Depth (`TONAL_DEPTH_WB_ENABLED`) and Light Wrap (`TONAL_LIGHT_WRAP_ENABLED`)
both read depth with their own permutation dimensions, but if `SceneDepth` is not bound,
they read an unbound/stale resource — producing corrupted output or a GPU device lost.
This is the same class of bug that caused original Tonal features to appear inactive unless
another was also on.

**Todos**

**TonalViewExtension.cpp**
- [ ] Compute `bool bNeedsDepth` as described in the Depth Binding Rule above
- [ ] Move `SceneDepth` SRV binding outside any single permutation's condition block
- [ ] Bind `SceneDepth` and `DepthSampler` whenever `bNeedsDepth` is true, regardless
  of which specific depth feature(s) are active
- [ ] Add an `ensure(bNeedsDepth)` guard in debug builds inside each depth-reading
  permutation's dispatch path to catch future regressions

---

### BUG-2 — Hue detection reads modified Result, not original signal

**Root cause:** If any feature runs before a hue-detection block and modifies `Result`,
the HSV extraction that follows operates on the wrong signal. Specifically:
- Skin Softener's hue detection is protected (runs first, `Result` is still `CenterColor.rgb`).
- Hue-Select Slots (new) would run at 5d, after Bleach Bypass and three Color Science steps
  have modified `Result`. A pixel's hue may have shifted enough that it misses the target
  range, silently failing — no error, just "feature seems weaker."

**Fix:** All hue-based detection (Skin Softener and all Hue-Select slots) must detect from
`CenterColor.rgb`, the original sampled value, not from `Result`. Only the grade
*application* uses `Result`. This is enforced by the multi-slot design below (slots
receive `CenterColor.rgb` for detection explicitly).

**Todos**

**TonalLDRFusion.usf**
- [ ] In the Skin Softener block, change hue/chroma extraction to operate on
  `CenterColor.rgb` explicitly (it currently does this correctly since `Result == CenterColor.rgb`
  at that point, but make it explicit and document the intent so it's not accidentally broken
  if the block order ever changes)
- [ ] All Hue-Select slot detection (see Feature A) uses `CenterColor.rgb` — enforce this
  in the slot loop with a named variable `float3 DetectColor = CenterColor.rgb`

---

### BUG-3 — Permutation/CVar sync review

**Root cause:** The original "feature not active unless another is on" was a permutation
mismatch — the compiled shader variant doesn't include a feature's code even though the
UB parameters are populated, because the permutation dimension was set incorrectly.

When adding two new permutation dimensions (`TONAL_DEPTH_WB_ENABLED`,
`TONAL_LIGHT_WRAP_ENABLED`), each must be verified end-to-end:
CVar value → `FFrameData` field → permutation dimension selection in `GetPermutationVector()`
→ compiled shader variant → UB parameter read.

**Todos**

**TonalViewExtension.cpp**
- [ ] For each new permutation dimension, add a comment block above the dimension-set
  line that names the CVar it mirrors: `// mirrors r.Tonal.DepthWB.Enabled`
- [ ] After adding all v2 permutations, write a single test level that enables each feature
  one at a time in isolation and verifies the expected visual change appears. This catches
  any CVar→permutation disconnect before shipping.
- [ ] Add `static_assert` or compile-time check that `NUM_PERMUTATION_DIMS == 7` to
  catch future accidental omissions

---

## Feature A — Hue-Select Slots (Multi-Slot Qualifier)

**What it does:** A multi-slot hue qualification system. Each slot independently selects
pixels by hue, saturation, and luminance range, then applies its own color grade
(tint/saturation/luma adjustment) to only those pixels. Slots are additive — all active
slots apply in sequence, each reading from `CenterColor.rgb` for detection. Up to 4 slots.

This is the secondary correction "qualifier" model from professional grading tools —
select a sky, grade it blue; select skin, warm it; select foliage, boost saturation;
select specular hotspots, cool them. Each independently, without touching the others.

**Pipeline stage:** LDR (5d), after Highlight Hue Shift (5c), before Subject Emphasis (6).
**Permutation:** None — inline `HueSelectSlotCount > 0` check.
**Debug mode:** 6 — composite of all active slot masks (each slot's mask added together,
clamped). Shows which pixels are being affected by any slot.

### Uniform buffer slot layout

Fixed array of 4 slots. Each slot packs into 4 `float4`s for alignment:

```
Pack 0: HueCenter, HueRange, SatMin, SatMax
Pack 1: LumaMin, LumaMax, Falloff, Strength
Pack 2: TintR, TintG, TintB, SatAdjust
Pack 3: HueRotate, LumaAdjust, _Pad, _Pad
```

`HueSelectSlotCount` (int, 0–4) controls how many slots the loop processes.
Slots beyond `SlotCount` are skipped entirely — zero cost.

### Shader algorithm per slot

```hlsl
float3 DetectColor = CenterColor.rgb;  // ALWAYS original — never Result

float MaxC = max(DetectColor.r, max(DetectColor.g, DetectColor.b));
float MinC = min(DetectColor.r, min(DetectColor.g, DetectColor.b));
float Chroma = MaxC - MinC;

float Hue = 0.0;
if (Chroma > 0.001)  // use TonalToHSV helper
    Hue = TonalToHSV(DetectColor).x;

float HueDist = min(abs(Hue - Slot.HueCenter), 1.0 - abs(Hue - Slot.HueCenter));
float HueMask = saturate(1.0 - HueDist / max(Slot.HueRange, 0.001));

float Sat = SafeRatio(Chroma, MaxC);
float SatMask = smoothstep(Slot.SatMin - Slot.Falloff, Slot.SatMin, Sat)
              * smoothstep(Slot.SatMax + Slot.Falloff, Slot.SatMax, Sat);

float DetectLuma = TonalLuminance(DetectColor);
float LumaMask = smoothstep(Slot.LumaMin - Slot.Falloff, Slot.LumaMin, DetectLuma)
               * smoothstep(Slot.LumaMax + Slot.Falloff, Slot.LumaMax, DetectLuma);

float SelectMask = HueMask * SatMask * LumaMask * Slot.Strength;

// Apply grade to Result (not DetectColor)
float3 Graded = Result;
float ResultLuma = TonalLuminance(Result);

// Tint (multiplicative)
Graded *= lerp(float3(1,1,1), float3(Slot.TintR, Slot.TintG, Slot.TintB), SelectMask);

// Saturation adjust
float GradedLuma = TonalLuminance(Graded);
Graded = GradedLuma + (Graded - GradedLuma) * (1.0 + Slot.SatAdjust * SelectMask);

// Luma adjust (exposure-style, multiplicative)
float LumaScale = 1.0 + Slot.LumaAdjust * SelectMask;
Graded *= max(LumaScale, 0.0);

Result = clamp(Graded, 0.0, 1.0);
```

Note: Hue rotation is omitted from the inline above for brevity — it requires an HSV→RGB
round-trip. Implement via `TonalHueRotate(float3 RGB, float Shift)` in `TonalCommon.ush`.

### Todos

**TonalCommon.ush**
- [ ] Add `float3 TonalToHSV(float3 RGB)` — returns `float3(H, S, V)`
  (refactor skin softener's existing inline hue math to call this)
- [ ] Add `float3 TonalFromHSV(float3 HSV)` — HSV→RGB for hue rotation
- [ ] Add `float3 TonalHueRotate(float3 RGB, float ShiftNormalized)` helper
  (converts to HSV, shifts H, converts back)

**TonalLDRFusion.usf**
- [ ] Refactor Skin Softener hue extraction to call `TonalToHSV`
- [ ] Add section 5d (Hue-Select Slots) after 5c:
  - `float3 DetectColor = CenterColor.rgb` — hardcoded, documented
  - Loop `for (int s = 0; s < TonalLDR.HueSelectSlotCount; s++)`
  - Per-slot: HSV from `DetectColor`, tri-mask, grade applied to `Result`
  - After loop: debug mode 6 composite mask output
- [ ] Update `DebugMode == 6` to accumulate `SelectMask` across all slots,
  show as `saturate(TotalMask)` greyscale

**FTonalHueSelectSlot (new C++ struct, TonalViewExtension.h)**
- [ ] Declare `struct FTonalHueSelectSlot` with all 16 float fields + padding
- [ ] Add `TStaticArray<FTonalHueSelectSlot, 4> HueSelectSlots` to `FFrameData`
- [ ] Add `int32 HueSelectSlotCount` to `FFrameData`
- [ ] Declare packed slot arrays in `FTonalLDRParameters` UB using `SHADER_PARAMETER_ARRAY`
  (4 float4 packs × 4 slots = 16 float4 entries total)

**TonalCVars.h / TonalCVars.cpp**
- [ ] `r.Tonal.HueSelect.SlotCount` — int 0..4, default 0
- [ ] No per-slot CVars — slot data is a UProperty array (see Settings below).
  CVars are impractical for array-of-struct data; UProperties are the right surface.
- [ ] Extend `r.Tonal.DebugView` tooltip to include mode 6

**TonalViewExtension.cpp**
- [ ] Read `r.Tonal.HueSelect.SlotCount` and clamp to 0–4
- [ ] Read slot data from `UTonalSettings::HueSelectSlots` array into `FFrameData`
- [ ] Pack into UB float4 arrays before dispatch

**TonalSettings.h / TonalSettings.cpp**
- [ ] Declare `USTRUCT() FTonalHueSelectSlotSettings` with all parameters as UPROPERTYs:
  `HueCenter` (float, 0-1), `HueRange` (float, 0.01-0.5), `SatMin/Max` (float),
  `LumaMin/Max` (float), `Falloff` (float), `Strength` (float),
  `Tint` (FLinearColor), `SatAdjust` (float, -1..1), `HueRotate` (float, -0.5..0.5),
  `LumaAdjust` (float, -1..1)
- [ ] Add `TArray<FTonalHueSelectSlotSettings> HueSelectSlots` to `UTonalSettings`
  with `UPROPERTY(EditAnywhere, Category="Hue Selection")`
- [ ] `ApplyToCVars()` — write `SlotCount`, pack slot data into frame data
- [ ] `ReadFromCVars()` — read count back (slot data lives in UProperty, not CVars)
- [ ] `Lerp()` — lerp each slot's float fields pairwise; if slot counts differ, lerp
  toward zero strength on the extra slots

**TonalPanel.cpp (Editor)**
- [ ] Add "Hue Selection" section
- [ ] Display active slots as a list: `[Slot 1] [Slot 2] [+] [-]`
- [ ] Per-slot expanded view:
  - Hue ring picker (circular 0–1 hue selector with range drag handles)
  - Sat range (min/max with feather)
  - Luma range (min/max with feather)
  - Strength slider
  - Tint: color wheel picker (maps to `FLinearColor Tint`)
  - Saturation Adjust: bipolar slider (-1 to +1)
  - Luma Adjust: bipolar slider (-1 to +1)
  - Hue Rotate: bipolar slider (-0.5 to +0.5)
- [ ] "+" adds a slot (max 4), "-" removes selected slot
- [ ] Add mode 6 to debug dropdown

---

## Feature B — Film Stock Emulation

**What it does:** Physically-motivated per-channel tone response modeled after real film
stocks. Red, green, and blue channels compress at different rates (independent shoulder
and toe per channel), which naturally produces the hue shifts characteristic of real film:
Kodak Vision3 highlights shift warm-orange because the blue channel shoulder compresses
more aggressively than red. Fuji Eterna shadows retain a slight blue-green cast from the
same mechanism. This cannot be reproduced with any combination of LDR color science tools
— the hue shift must emerge from pre-tonemapper per-channel imbalance.

**Pipeline stage:** HDR, Pass 2 — after Shaper, before Midtone Glow.
**Implementation:** New compute shader `TonalFilmStock.usf`, conditionally dispatched.
Zero cost when disabled (no dispatch, no permutations).

**Important:** Film Stock reshapes the signal before ACES runs, which shifts the LDR
output that all LDR features operate on. Treat Film Stock as a scene-level baseline —
set it before tuning any LDR sliders. The editor panel should show a banner when Film
Stock is active: *"HDR signal modified — LDR thresholds may need adjustment."*

### Shader notes
- Per-channel sigmoid: for each channel, apply shoulder and toe independently using the
  same Hill/power math as `ApplyHDRShaper`, but without the luma-ratio preservation step.
  The hue shift emerges naturally from this asymmetry — no explicit hue rotation.
- Reinhard-protect input before processing to handle extreme HDR values gracefully:
  `float3 Protected = InColor / (1.0 + TonalLuminance(InColor))` — deprotect after.
- Blend back to input: `lerp(InColor, Result, FilmStockBlend)` for partial-strength looks.
- Debug mode 1 (own CVar): per-channel delta heatmap — R channel = red shift,
  G channel = green shift, B channel = blue shift. Use `abs(Out[ch] - In[ch]) * 4.0`.

### Named Presets (ship as UTonalPreset data assets)
| Preset | R Shoulder | G Shoulder | B Shoulder | R Toe | G Toe | B Toe | Character |
|---|---|---|---|---|---|---|---|
| Vision3 500T | 0.35 | 0.40 | 0.55 | 0.10 | 0.08 | 0.06 | Warm highlights, neutral shadows |
| Vision3 250D | 0.30 | 0.38 | 0.50 | 0.06 | 0.06 | 0.04 | Cooler, cleaner |
| Eterna 500 | 0.28 | 0.32 | 0.40 | 0.12 | 0.10 | 0.08 | Soft, slight green-shadow cast |
| Digital Neutral | 0.40 | 0.40 | 0.40 | 0.05 | 0.05 | 0.05 | Symmetric — pleasant shoulder only |

### Todos

**TonalCVars.h / TonalCVars.cpp**
- [ ] `r.Tonal.FilmStock.Enabled` — int 0/1, default 0
- [ ] `r.Tonal.FilmStock.Blend` — float 0..1, default 1.0
- [ ] `r.Tonal.FilmStock.ShoulderR/G/B` — float 0..1, defaults 0.35/0.40/0.55
- [ ] `r.Tonal.FilmStock.ToeR/G/B` — float 0..0.5, defaults 0.10/0.08/0.06
- [ ] `r.Tonal.FilmStock.DebugMode` — int 0/1, default 0 (own CVar, separate from LDR)
- [ ] `r.Tonal.FilmStock.Preset` — int 0..4, default 0 (0=Custom; writing non-zero copies
  preset values into the above CVars)

**TonalViewExtension.h / TonalViewExtension.cpp**
- [ ] `FTonalFilmStockParameters` UB struct: `ShoulderR/G/B`, `ToeR/G/B`, `Blend`, `DebugMode`
- [ ] `FTonalFilmStockCS` global compute shader class, entry `MainCS`
- [ ] Add film stock fields to `FFrameData`; read CVars in `BeginRenderViewFamily`
- [ ] `RenderHDRFilmStock(FRDGBuilder&, FRDGTextureRef Input, FRDGTextureRef& Output)`
  — called after `RenderHDRShaper`, output feeds into `RenderHDRMidtoneGlow`
- [ ] Skip dispatch entirely when `!bFilmStockEnabled`

**Shaders/TonalFilmStock.usf** (new file)
- [ ] Standard resource declarations matching `TonalHDRShaper.usf` pattern
- [ ] `ApplyFilmStockPerChannel(float3 In, float3 Shoulder, float3 Toe) → float3`
  with Reinhard protect/deprotect
- [ ] `[numthreads(8,8,1)] MainCS` — per-channel curve, blend, sanitize
- [ ] Debug mode 1: per-channel delta heatmap

**Content/Presets/** (new UTonalPreset data assets)
- [ ] `FS_Vision3_500T`, `FS_Vision3_250D`, `FS_Eterna_500`, `FS_Digital_Neutral`

**TonalSettings.h / TonalSettings.cpp**
- [ ] `FilmStock*` UPROPERTYs, `Category="Film Stock (HDR)"`
- [ ] `ApplyFilmStockPreset(EFilmStockPreset)` helper
- [ ] Add to sync/lerp methods

**TonalPanel.cpp (Editor)**
- [ ] Add "Film Stock (HDR)" section above "Midtone Glow (HDR)"
- [ ] Active-state banner: yellow info box reading *"HDR signal modified — LDR thresholds
  may need adjustment"* when `FilmStock.Enabled == 1`
- [ ] Preset dropdown: Custom / Vision3 500T / Vision3 250D / Eterna 500 / Digital Neutral
- [ ] Blend slider always visible
- [ ] Expandable "Advanced": per-channel Shoulder/Toe sliders (shown when Preset = Custom)
- [ ] Separate Film Stock debug toggle (not the main DebugView dropdown)

---

## Feature C — Bleach Bypass

**What it does:** Emulates photochemical bleach bypass: lifted blacks, crushed saturation,
elevated contrast. Produces the gritty high-contrast look of *28 Days Later*,
*Saving Private Ryan*, *Se7en*. Fully parametric blend (0=off, 1=full bypass, 0.3=subtle).

**Pipeline stage:** LDR, section 4 — runs before Color Science (5a–5d). Bleach bypass is a
whole-image tone foundation; Color Science then refines on top of the bypassed result.
**Permutation:** None — inline `BleachBypassStrength > 0.001` check.

### Shader notes
- Capture `float3 OriginalColor = Result` before processing.
- Desaturation: `float3 Grey = float3(Luma, Luma, Luma)`; lerp `Result` toward `Grey`
  by `DesatAmount * BleachBypassStrength`.
- Shadow lift: `float ShadowMask = pow(saturate(1.0 - Luma / 0.5), 2.0)`;
  `Result += LiftAmount * BleachBypassStrength * ShadowMask`.
- Log-space contrast (avoids color shift from multiplicative contrast):
  ```
  float LogLuma = log2(max(TonalLuminance(Result), 0.001));
  float ContrastLuma = exp2(LogLuma * ContrastBoost);
  Result *= ContrastLuma / max(TonalLuminance(Result), 0.001);
  ```
- Final blend back: `Result = lerp(OriginalColor, Result, BleachBypassStrength)`.
- Clamp to [0, 1].

### Todos

**TonalCVars.h / TonalCVars.cpp**
- [ ] `r.Tonal.BleachBypass.Strength` — float 0..1, default 0
- [ ] `r.Tonal.BleachBypass.DesatAmount` — float 0..1, default 0.5
- [ ] `r.Tonal.BleachBypass.LiftAmount` — float 0..0.3, default 0.08
- [ ] `r.Tonal.BleachBypass.ContrastBoost` — float 1.0..2.0, default 1.3

**TonalViewExtension.h / TonalViewExtension.cpp**
- [ ] Add fields to `FFrameData` and `FTonalLDRParameters`
- [ ] Read CVars in `BeginRenderViewFamily`

**TonalLDRFusion.usf**
- [ ] Add section 4 (Bleach Bypass) after Split Tone (3), before Color Science (5a):
  - `float3 OriginalColor = Result` capture
  - Desaturation, shadow lift, log-space contrast, final blend
  - Clamp [0, 1]

**TonalSettings.h / TonalSettings.cpp**
- [ ] `BleachBypass*` UPROPERTYs, `Category="Bleach Bypass"`
- [ ] Add to sync/lerp

**TonalPanel.cpp (Editor)**
- [ ] Add "Bleach Bypass" section (between Split Tone and Color Science)
- [ ] Strength slider as primary control
- [ ] Expandable "Advanced": Desat Amount, Lift Amount, Contrast Boost

---

## Feature D — Color Temperature by Depth

**What it does:** Applies independent white balance corrections to near, mid, and far depth
zones via the scene depth buffer. Near zone can be warm (tungsten interior), far zone cool
(daylight through windows). UE5 has no per-depth color temperature control.

**Pipeline stage:** LDR, section 8 — after Depth Fade, before Light Wrap. Shares the depth
read context; see depth deduplication note in cross-cutting work.
**Permutation:** `TONAL_DEPTH_WB_ENABLED` — eliminates depth reads when disabled.
**Debug mode:** 7 (zone mask: green=near, yellow=mid, red=far).

### Shader notes
- Depth linearization: reuse exact formula from Depth Fade block (lines 237–239 in
  `TonalLDRFusion.usf`). When both `TONAL_DEPTH_FADE_ENABLED` and `TONAL_DEPTH_WB_ENABLED`
  are active, compute `LinearDepth` once under a combined `#if` guard (see cross-cutting).
- Zone blend:
  ```
  float NearToMid = smoothstep(NearDist, MidDist, LinearDepth);
  float MidToFar  = smoothstep(MidDist,  FarDist,  LinearDepth);
  float3 WB = lerp(WB_Near, lerp(WB_Mid, WB_Far, MidToFar), NearToMid);
  Result *= WB;
  Result = clamp(Result, 0.0, 1.0);
  ```
- RGB multipliers directly in the UB (not Kelvin). Kelvin→RGB conversion lives in
  `UTonalSettings` helper methods on the C++ side.
- Debug mode 7: `float3(1.0 - NearToMid, 1.0 - MidToFar, MidToFar)` for zone visualization.

### Todos

**TonalCVars.h / TonalCVars.cpp**
- [ ] `r.Tonal.DepthWB.Enabled` — int 0/1, default 0
- [ ] `r.Tonal.DepthWB.NearDist` — float cm, default 500
- [ ] `r.Tonal.DepthWB.MidDist` — float cm, default 5000
- [ ] `r.Tonal.DepthWB.FarDist` — float cm, default 50000
- [ ] `r.Tonal.DepthWB.NearR/G/B` — float 0.5..2.0, default 1.0
- [ ] `r.Tonal.DepthWB.MidR/G/B` — float 0.5..2.0, default 1.0
- [ ] `r.Tonal.DepthWB.FarR/G/B` — float 0.5..2.0, default 1.0
- [ ] Extend `r.Tonal.DebugView` tooltip to include mode 7

**TonalViewExtension.h / TonalViewExtension.cpp**
- [ ] Add fields to `FFrameData` and `FTonalLDRParameters`
- [ ] Add `TONAL_DEPTH_WB_ENABLED` to permutation enum and selection logic
- [ ] Update `bNeedsDepth` to include `bDepthWBEnabled` (BUG-1)
- [ ] Read CVars in `BeginRenderViewFamily`

**TonalLDRFusion.usf**
- [ ] Add `TONAL_DEPTH_WB_ENABLED` block (section 8)
- [ ] Linearize depth (or share with Depth Fade — see cross-cutting deduplication)
- [ ] Zone blend, RGB multiply, clamp
- [ ] Debug mode 7

**TonalSettings.h / TonalSettings.cpp**
- [ ] `DepthWB*` UPROPERTYs, `Category="Depth Color Temperature"`
- [ ] `SetNearKelvin(float K)` / `SetFarKelvin(float K)` helpers (Kelvin→RGB approx,
  writes to R/G/B CVars)
- [ ] Add to sync/lerp

**TonalPanel.cpp (Editor)**
- [ ] "Depth Color Temperature" section
- [ ] Zone distance sliders (Near / Mid / Far)
- [ ] Per-zone: Kelvin picker that maps to RGB internally, plus optional direct R/G/B toggle
- [ ] Add mode 7 to debug dropdown

---

## Feature E — Local Tone Mapping

**What it does:** Spatially redistributes luminance to recover shadow detail in high-contrast
scenes without flattening. Decomposes the image into a bilateral base layer and a
high-frequency detail layer; lifts the base in shadow regions; restores detail on top.

UE5's Local Exposure (5.1+) runs in HDR, scales exposure per region, and does not decompose
base+detail — so shadow detail recovery without halos is not achievable with it. This
operates in LDR post-tonemap and fills that gap.

**Pipeline stage:** LDR pre-pass — two dispatches before `TonalLDRFusion`. Conditionally
dispatched; zero cost when disabled.
**Why a pre-pass:** The bilateral downsample writes a 1/4-resolution RDG texture that must
be read in a subsequent dispatch. The fusion shader is single-dispatch and cannot accommodate
this dependency inline.

### Pass 1 — TonalLocalToneBase.usf
Dispatch at `ceil(ViewportSize / 4)`. 9-tap bilateral downsample:
- Bilateral weight per neighbor: `exp(-(LumaDiff^2) / SigmaLuma)` where
  `LumaDiff = abs(CenterLuma - SampleLuma) / (CenterLuma + 0.1)`
- Output: `float4` base layer at 1/4 resolution (`PF_FloatRGBA`)

### Pass 2 — TonalLocalToneReconstruct.usf
Dispatch at full `ViewportSize`. Shadow lift + detail reconstruct:
- Sample `BaseTexture` at full-res UV (bilinear from 1/4-res — natural upsample)
- `float3 Detail = FullResColor - BaseLayer` (high-frequency residual)
- `float BaseLuma = TonalLuminance(Base)`
- `float LiftMask = exp(-BaseLuma * BaseLuma / (2 * ShadowThreshold * ShadowThreshold))`
  (Gaussian, peaks at luma=0)
- `float3 LiftedBase = Base * (1.0 + LiftMask * LiftAmount)`
- `LiftedBase = min(LiftedBase, Base * 1.25)` — hard cap, prevents highlight blowout
- `Result = clamp(LiftedBase + Detail, 0.0, 1.0)`
- Debug mode 1: output base layer; Debug mode 2: output `Detail * 0.5 + 0.5`

### Todos

**TonalCVars.h / TonalCVars.cpp**
- [ ] `r.Tonal.LocalTone.Enabled` — int 0/1, default 0
- [ ] `r.Tonal.LocalTone.LiftAmount` — float 0..1, default 0.3
- [ ] `r.Tonal.LocalTone.ShadowThreshold` — float 0.05..0.5, default 0.2
- [ ] `r.Tonal.LocalTone.SigmaLuma` — float 0.01..1.0, default 0.1
- [ ] `r.Tonal.LocalTone.DebugMode` — int 0/1/2, default 0 (own CVar)

**TonalViewExtension.h / TonalViewExtension.cpp**
- [ ] `FTonalLocalToneParameters` UB: `LiftAmount`, `ShadowThreshold`, `SigmaLuma`,
  `DebugMode`, `ViewportSize`, `InvViewportSize`
- [ ] `FTonalLocalToneBaseCS` compute shader class
- [ ] `FTonalLocalToneReconstructCS` compute shader class
- [ ] `RenderLDRLocalTone(FRDGBuilder&, FRDGTextureRef SceneColor, FRDGTextureRef& OutResult)`
  — called before existing LDR fusion pass; creates 1/4-res RDG texture, dispatches both CS,
  output texture becomes LDR fusion input
- [ ] Add local tone fields to `FFrameData`; read CVars in `BeginRenderViewFamily`
- [ ] Skip both dispatches when `!bLocalToneEnabled`

**Shaders/TonalLocalToneBase.usf** (new file)
- [ ] Bilateral 9-tap downsample (standard resource declarations)

**Shaders/TonalLocalToneReconstruct.usf** (new file)
- [ ] `BaseTexture` (1/4-res SRV) + `InputTexture` (full-res SRV) + `OutputTexture`
- [ ] Shadow lift + detail reconstruct
- [ ] Debug modes 1 and 2

**TonalSettings.h / TonalSettings.cpp**
- [ ] `LocalTone*` UPROPERTYs, `Category="Local Tone Mapping"`
- [ ] Panel tooltip note: *"Works alongside UE5 Local Exposure (HDR). This operates in LDR
  for detail-preserving shadow recovery."*
- [ ] Add to sync/lerp

**TonalPanel.cpp (Editor)**
- [ ] "Local Tone Mapping" section
- [ ] Lift Amount, Shadow Threshold, Sigma Luma sliders
- [ ] Separate "LTM Debug" dropdown (Off / Base Layer / Detail Layer)

---

## Feature F — Screen-Space Light Wrap

**What it does:** Bleeds bright background colors onto the edges of foreground subjects.
Produces the natural rim-light integration that makes rendered objects feel grounded in
their environment — particularly valuable for virtual production (LED volume) and CG
integration in archviz/games. UE5 has no equivalent.

**Pipeline stage:** LDR, section 9 — last effect before output. Wraps the final composed
image.
**Permutation:** `TONAL_LIGHT_WRAP_ENABLED` — eliminates depth reads and multi-tap blur
when disabled.
**Debug mode:** 8 (edge mask: white=edge, black=interior).

### Shader notes
- **Edge detection (depth-based):** 5-tap cross depth sample (center + L/R/U/D), linearize
  each. Max absolute delta vs center, scaled by `EdgeSensitivity`. Soft-clip to [0, 1].
- **Background sample (blurred):** 5-tap blur on `Result` at `WrapBlurRadius` pixel offset.
  Weighted average gives blurred background color spreading outward.
- **Composite:** `Result = Result + BlurredColor * EdgeStrength * WrapIntensity`
- **Bright guard:** `WrapContrib = min(WrapContrib, 0.3)` — prevents blowing out bright
  foreground pixels at strong settings.
- All sample UVs clamped to `[InputUVMin, InputUVMax]` to prevent viewport-edge artifacts.

### Todos

**TonalCVars.h / TonalCVars.cpp**
- [ ] `r.Tonal.LightWrap.Enabled` — int 0/1, default 0
- [ ] `r.Tonal.LightWrap.Intensity` — float 0..1, default 0.15
- [ ] `r.Tonal.LightWrap.BlurRadius` — float 1..16, default 4
- [ ] `r.Tonal.LightWrap.EdgeSensitivity` — float 0.1..50, default 5.0
- [ ] `r.Tonal.LightWrap.UseDepthEdge` — int 0/1, default 1
- [ ] Extend `r.Tonal.DebugView` tooltip to include mode 8

**TonalViewExtension.h / TonalViewExtension.cpp**
- [ ] Add fields to `FFrameData` and `FTonalLDRParameters`
- [ ] Add `TONAL_LIGHT_WRAP_ENABLED` to permutation enum and selection logic
- [ ] Update `bNeedsDepth` to include `bLightWrapEnabled && bLightWrapUseDepth` (BUG-1)
- [ ] Read CVars in `BeginRenderViewFamily`

**TonalLDRFusion.usf**
- [ ] Add `TONAL_LIGHT_WRAP_ENABLED` block (section 9):
  - 5-tap depth reads (center + L/R/U/D), linearize each
  - Max-delta edge strength, scaled and saturated
  - 5-tap `Result` blur at `WrapBlurRadius`
  - Additive composite with bright-pixel guard and final clamp
  - Debug mode 8: edge mask as greyscale

**TonalSettings.h / TonalSettings.cpp**
- [ ] `LightWrap*` UPROPERTYs, `Category="Light Wrap"`
- [ ] Add to sync/lerp

**TonalPanel.cpp (Editor)**
- [ ] "Light Wrap" section (last section before debug)
- [ ] Intensity, Blur Radius, Edge Sensitivity sliders
- [ ] "Depth Edge" / "Luminance Edge" toggle
- [ ] Add mode 8 to debug dropdown

---

## Cross-Cutting Work

Apply once across all features.

- [ ] **`TonalViewExtension.cpp`** — Implement `bNeedsDepth` (BUG-1):
  ```cpp
  bool bNeedsDepth = FrameData.bDepthFadeEnabled
                  || FrameData.bDepthWBEnabled
                  || (FrameData.bLightWrapEnabled && FrameData.bLightWrapUseDepth);
  ```
  Bind `SceneDepth` SRV unconditionally when `bNeedsDepth`. Remove any per-feature
  binding of this resource.

- [ ] **`TonalLDRFusion.usf`** — Depth linearization deduplication: declare `float LinearDepth`
  once under `#if TONAL_DEPTH_FADE_ENABLED || TONAL_DEPTH_WB_ENABLED || TONAL_LIGHT_WRAP_ENABLED`.
  Both Depth Fade and Depth WB blocks read from this shared variable instead of recomputing.

- [ ] **`TonalCommon.ush`** — Add `TonalToHSV`, `TonalFromHSV`, `TonalHueRotate` helpers.
  Refactor Skin Softener hue extraction to use `TonalToHSV` (BUG-2).

- [ ] **`TonalViewExtension.cpp`** — Permutation sync review (BUG-3): for each of the 7
  permutation dimensions, add comment naming the CVar it mirrors. Add `static_assert` or
  compile-time check for `NUM_PERMUTATION_DIMS == 7`.

- [ ] **`TonalCVars.cpp`** — Update `r.Tonal.DebugView` description to list all modes 1–8.

- [ ] **`TonalPanel.cpp`** — Update master debug mode dropdown to include modes 6–8.

- [ ] **Preset assets** — Create at least one `UTonalPreset` that exercises each new feature
  with sensible defaults (QA coverage of all new code paths).

- [ ] **`Tonal.uplugin`** — Bump `VersionName` from `"1.0"` to `"2.0"`. Keep
  `IsBetaVersion: true` until all features and bug fixes are verified.
