# Lucent DOF Resolution Quality & Performance — Implementation Todo

**Date:** 2026-03-05
**Branch:** AttemptTwoOnHeadUpdate
**Session context:** User observed full-res gather looks significantly better than half-res, specifically due to TAA shimmer artifacts being worse at half-res. Needs: improved half-res quality, better full-res perf, 2/3 res option.

---

## Root Cause Summary

TAA runs BEFORE Lucent's DOF. TAA leaves shimmer on high-frequency surfaces. The chain is:

```
RenderDOFPad (bilinear downsample) → DispatchGather (Reinhard gather) → RenderDOFTemporal → RenderDOFUpsample
```

**At full-res**: each gather thread sees one full-res pixel. Reinhard suppression sees the TAA spike at full intensity and clamps it. Temporal suppresses any residual inconsistency.

**At half-res**: the pad pass box-averages 4 full-res pixels (via a SINGLE bilinear tap) into each gather texel BEFORE Reinhard sees them. TAA spikes are diluted into neighboring texels. The diluted smear is then spatial-frequency-stable (looks the same frame-to-frame), so temporal doesn't flag it as an outlier. Result: shimmer is smeared rather than suppressed.

**Key file — the single bilinear tap is the core problem:**
- `Plugins/Lucent/Shaders/LucentDOFPad.usf` line 46: `SceneColor.SampleLevel(InputSampler, SourceUV, 0)` — a single hardware bilinear tap. This IS a box filter average at `GatherScale=2`.

---

## Changes Needed (Priority Order)

---

### [1] HIGHEST PRIORITY — Karis-weighted 4-tap downsample in LucentDOFPad.usf

**File:** `Plugins/Lucent/Shaders/LucentDOFPad.usf`
**What:** Replace the single bilinear tap with a 4-tap box filter where each tap is weighted by `1/(1+Luma)` (Karis weight). This suppresses TAA outliers BEFORE the gather mixes them into neighboring texels.
**Only active at GatherScale > 1** — at GatherScale=1 the 4 taps would land on the same pixel so a single tap is correct.

**Current code (lines 40-48):**
```hlsl
[numthreads(8, 8, 1)]
void MainCS(uint3 DispatchThreadId : SV_DispatchThreadID)
{
    uint2 PixelPos = DispatchThreadId.xy;
    if (PixelPos.x >= (uint)PaddedSize.x || PixelPos.y >= (uint)PaddedSize.y)
        return;

    float2 ViewportUV = (float2(PixelPos) - PadPixels + 0.5) * GatherScale / ViewportSize;
    float2 ClampedUV = clamp(ViewportUV, 0.0, 1.0);
    float2 SourceUV  = lerp(InputUVMin, InputUVMax, ClampedUV);

    OutputTexture[PixelPos] = SceneColor.SampleLevel(InputSampler, SourceUV, 0);
}
```

**Target code:**
```hlsl
[numthreads(8, 8, 1)]
void MainCS(uint3 DispatchThreadId : SV_DispatchThreadID)
{
    uint2 PixelPos = DispatchThreadId.xy;
    if (PixelPos.x >= (uint)PaddedSize.x || PixelPos.y >= (uint)PaddedSize.y)
        return;

    float2 ViewportUV = (float2(PixelPos) - PadPixels + 0.5) * GatherScale / ViewportSize;

    // At GatherScale == 1: single tap (no downsampling, 4 taps would land on same pixel).
    // At GatherScale > 1: 4-tap Karis-weighted box filter.
    // Karis weighting 1/(1+Luma) suppresses HDR outliers (TAA shimmer, hot specular)
    // before they can be smeared into neighboring gather texels by the gather kernel.
    if (GatherScale <= 1.0 + 1e-3)
    {
        float2 ClampedUV = clamp(ViewportUV, 0.0, 1.0);
        float2 SourceUV  = lerp(InputUVMin, InputUVMax, ClampedUV);
        OutputTexture[PixelPos] = SceneColor.SampleLevel(InputSampler, SourceUV, 0);
        return;
    }

    // 4-tap Karis-weighted downsample.
    // Tap offsets are at ±0.25 texels in source (full-res) UV space.
    float2 FullResTexelSize = (InputUVMax - InputUVMin) / ViewportSize;
    float2 TapOffset = FullResTexelSize * 0.5;

    float3 AccumColor  = float3(0, 0, 0);
    float  AccumWeight = 0.0;

    [unroll]
    for (int dy = 0; dy <= 1; dy++)
    {
        [unroll]
        for (int dx = 0; dx <= 1; dx++)
        {
            float2 TapUV = clamp(ViewportUV + float2(dx - 0.5, dy - 0.5) * (GatherScale / ViewportSize), 0.0, 1.0);
            float2 SourceUV = lerp(InputUVMin, InputUVMax, TapUV);
            float3 TapColor = SceneColor.SampleLevel(InputSampler, SourceUV, 0).rgb;

            // Karis weight: suppress HDR outlier influence during downsampling.
            float Luma = dot(TapColor, float3(0.2126, 0.7152, 0.0722));
            float W = 1.0 / (1.0 + Luma);

            AccumColor  += TapColor * W;
            AccumWeight += W;
        }
    }

    float3 Result = AccumColor / max(AccumWeight, 1e-6);
    OutputTexture[PixelPos] = float4(Result, 1.0);
}
```

**Impact:** This is the most impactful single change. TAA spikes are attenuated before the gather spreads them.

---

### [2] HIGH PRIORITY — 2/3 Resolution Option

**Goal:** Add `GatherScale=3` as a 4th option meaning "gather at 2/3 resolution" (~2.25× faster than full-res, much better quality than half-res since the 2/3 downsample averages only a 1.5×1.5 block instead of a 2×2 block).

#### 2a. LucentCVars.cpp — expand the CVar text
**File:** `Plugins/Lucent/Source/Lucent/LucentCVars.cpp` lines 218-226

**Current:**
```cpp
LUCENT_API TAutoConsoleVariable<int32> CVarLucentDOFGatherScale(
    TEXT("r.Lucent.DOF.GatherScale"),
    0,
    TEXT("DOF gather resolution preset (index).\n")
    TEXT("  0: Full resolution (default — maximum quality)\n")
    TEXT("  1: Half resolution (~4x cheaper; depth-aware upsample restores full-res)\n")
    TEXT("  2: Quarter resolution (~16x cheaper; suitable for large CoC / cinematic renders)"),
    ECVF_RenderThreadSafe
);
```

**Target:**
```cpp
LUCENT_API TAutoConsoleVariable<int32> CVarLucentDOFGatherScale(
    TEXT("r.Lucent.DOF.GatherScale"),
    0,
    TEXT("DOF gather resolution preset (index).\n")
    TEXT("  0: Full resolution (default — maximum quality)\n")
    TEXT("  1: Half resolution (~4x cheaper; depth-aware upsample restores full-res)\n")
    TEXT("  2: Quarter resolution (~16x cheaper; suitable for large CoC / cinematic renders)\n")
    TEXT("  3: Two-thirds resolution (~2.25x cheaper; best quality-per-cost tradeoff)"),
    ECVF_RenderThreadSafe
);
```

#### 2b. LucentViewExtension.cpp — change GatherRes calculation
**File:** `Plugins/Lucent/Source/Lucent/LucentViewExtension.cpp` line 652

**Current:**
```cpp
{ static const int32 GScaleTable[] = {1, 2, 4}; NewData.DOFGatherScale = GScaleTable[FMath::Clamp(CVarLucentDOFGatherScale.GetValueOnGameThread(), 0, 2)]; }
```

**Target:** Replace the lookup table with a stored preset index. The GatherRes calculation moves to `RenderDOFStage`.
Change `FFrameData::DOFGatherScale` to store the preset index (0-3), not the divisor.

```cpp
NewData.DOFGatherScale = FMath::Clamp(CVarLucentDOFGatherScale.GetValueOnGameThread(), 0, 3);
```

#### 2c. LucentViewExtension.cpp — update RenderDOFStage GatherRes calculation
**File:** `Plugins/Lucent/Source/Lucent/LucentViewExtension.cpp` lines 1776-1784

**Current:**
```cpp
// Half/quarter-res gather settings from frame data.
const int32 GatherScale = FMath::Max(FD.DOFGatherScale, 1);

// Scale MaxBlurRadius to gather-res pixel space first (before computing PadPixels).
DOFParams.MaxBlurRadius /= static_cast<float>(GatherScale);

// GatherRes: the resolution at which bokeh gathering is performed.
const FIntPoint GatherRes = FIntPoint(ViewportSize.X / GatherScale, ViewportSize.Y / GatherScale);
```

**Target:**
```cpp
// Gather resolution from preset index.
// Preset 0=full, 1=half, 2=quarter, 3=two-thirds.
FIntPoint GatherRes;
float GatherScaleFloat;
{
    const int32 Preset = FMath::Clamp(FD.DOFGatherScale, 0, 3);
    if (Preset == 3)
    {
        // Two-thirds: explicit size, rounded to even to avoid half-texel seam on upsample.
        GatherRes.X = FMath::Max(FMath::RoundToInt(ViewportSize.X * 2.0f / 3.0f) & ~1, 2);
        GatherRes.Y = FMath::Max(FMath::RoundToInt(ViewportSize.Y * 2.0f / 3.0f) & ~1, 2);
    }
    else
    {
        const int32 Divisor = (Preset == 0) ? 1 : (Preset == 1) ? 2 : 4;
        GatherRes.X = ViewportSize.X / Divisor;
        GatherRes.Y = ViewportSize.Y / Divisor;
    }
    // Float scale used by pad/upsample shaders — may be non-integer for 2/3 mode.
    GatherScaleFloat = static_cast<float>(ViewportSize.X) / static_cast<float>(GatherRes.X);
}

// Scale MaxBlurRadius to gather-res pixel space.
DOFParams.MaxBlurRadius /= GatherScaleFloat;
```

#### 2d. LucentViewExtension.cpp — update all GatherScale usages to GatherScaleFloat
Search and replace the remaining `GatherScale` variable uses (below line 1784 in `RenderDOFStage`) with `GatherScaleFloat`:
- `DOFParams.PaddedWidth/Height` calculations (already use `GatherRes` so no change)
- `RenderDOFPad(... PadPixels, GatherScale)` → `RenderDOFPad(... PadPixels, GatherScaleFloat)` — but `RenderDOFPad` signature takes `int32 GatherScale`, needs to become `float`
- `MaybeUpsample` condition: `if (GatherScale <= 1)` → `if (GatherScaleFloat <= 1.0f + 1e-3f)`
- `RenderDOFUpsample(... GatherRes, GatherScale ...)` → pass `GatherScaleFloat`

#### 2e. LucentViewExtension.h/.cpp — update RenderDOFPad signature
**File:** `Plugins/Lucent/Source/Lucent/LucentViewExtension.h` — `RenderDOFPad` declaration
**File:** `Plugins/Lucent/Source/Lucent/LucentViewExtension.cpp` — `RenderDOFPad` implementation

Change `int32 GatherScale` parameter to `float GatherScale` in both. The shader already receives it as a `float`.

**Current header declaration:**
```cpp
FRDGTextureRef RenderDOFPad(
    FRDGBuilder& GraphBuilder,
    FScreenPassTexture SceneColor,
    int32 PadPixels,
    int32 GatherScale);
```

**Target:**
```cpp
FRDGTextureRef RenderDOFPad(
    FRDGBuilder& GraphBuilder,
    FScreenPassTexture SceneColor,
    int32 PadPixels,
    float GatherScale);
```

**Current implementation line 1533:**
```cpp
FRDGTextureRef FLucentViewExtension::RenderDOFPad(
    FRDGBuilder& GraphBuilder,
    FScreenPassTexture SceneColor,
    int32 PadPixels,
    int32 GatherScale)
{
    const FIntPoint ViewportSize = SceneColor.ViewRect.Size();
    const FIntPoint GatherRes  = FIntPoint(ViewportSize.X / GatherScale, ViewportSize.Y / GatherScale);
```

**Target:**
```cpp
FRDGTextureRef FLucentViewExtension::RenderDOFPad(
    FRDGBuilder& GraphBuilder,
    FScreenPassTexture SceneColor,
    int32 PadPixels,
    float GatherScale)
{
    const FIntPoint ViewportSize = SceneColor.ViewRect.Size();
    // Round to nearest int; RenderDOFStage already computed GatherRes, but we recompute
    // here from GatherScaleFloat for self-consistency (only affects the pad texture size).
    const FIntPoint GatherRes = FIntPoint(
        FMath::Max(FMath::RoundToInt(ViewportSize.X / GatherScale), 1),
        FMath::Max(FMath::RoundToInt(ViewportSize.Y / GatherScale), 1));
```

**Note:** `FFrameData::DOFGatherScale` is currently `int32`. After this change it stores a preset index 0-3 which is still `int32`. No struct change needed.

---

### [3] MEDIUM PRIORITY — Temporal Neighbourhood Variance Clipping

**File:** `Plugins/Lucent/Shaders/LucentDOFTemporal.usf`
**What:** Replace the luminance-diff biased blend with a **colour AABB neighbourhood clamp**. Clamp the reprojected history to the bounding box of the current gather's 3×3 neighbourhood before blending. This prevents stale history from propagating stable-but-wrong colours (half-res smear) without requiring the pixel to flicker frame-to-frame to be detected.

**Current blend logic (lines 70-87):**
```hlsl
float CurrentLuma = dot(Current.rgb, float3(0.2126, 0.7152, 0.0722));
float HistoryLuma = dot(History.rgb, float3(0.2126, 0.7152, 0.0722));
float LumaDiff    = abs(CurrentLuma - HistoryLuma) / max(HistoryLuma, 0.1);

float BlendFactor = lerp(BlendMin, BlendMax,
                         saturate(1.0 - LumaDiff / max(LumaThreshold, 0.01)));

OutputTexture[PixelPos] = lerp(History, Current, BlendFactor);
```

**Target code:**
```hlsl
// Neighbourhood AABB: sample 3×3 gather texels around the current pixel.
// Clamp history into the colour bounding box before blending.
// This suppresses stale half-res smear that is spatially stable (same every frame)
// without requiring flickering frame-to-frame for detection.
float3 NeighbourMin = Current.rgb;
float3 NeighbourMax = Current.rgb;

[unroll]
for (int ny = -1; ny <= 1; ny++)
{
    [unroll]
    for (int nx = -1; nx <= 1; nx++)
    {
        if (nx == 0 && ny == 0) continue;
        float2 NUV = GatherUV + float2(nx, ny) / GatherResSize;
        float3 N = CurrentGather.SampleLevel(BilinearSampler, NUV, 0).rgb;
        NeighbourMin = min(NeighbourMin, N);
        NeighbourMax = max(NeighbourMax, N);
    }
}

// Clamp history to neighbourhood AABB.
float3 ClampedHistory = clamp(History.rgb, NeighbourMin, NeighbourMax);

// Blend: fixed BlendMax (no luminance-diff tuning needed after AABB clamp).
OutputTexture[PixelPos] = float4(lerp(ClampedHistory, Current.rgb, BlendMax), Current.a);
```

**Additional changes needed for this:**
- The `BlendMin`, `LumaThreshold` shader parameters become unused — they can be removed (C++ struct + shader), or kept as dead code until confirmed working.
- `BlendMax` becomes the single blend factor (~0.1 is a good default for temporal ghosting resistance).

**C++ changes:** In `FLucentDOFTemporalCS::FParameters` (LucentViewExtension.cpp), remove `BlendMin` and `LumaThreshold` parameters (or keep them, they're cheap).

---

### [4] LOWER PRIORITY — Adaptive Ring Count in LucentDOF.usf

**File:** `Plugins/Lucent/Shaders/LucentDOF.usf`
**What:** Instead of using the global `RingCount` for every pixel, clamp it based on `CenterCoC`. Small CoC (2px) only needs 1-2 rings; full rings only needed at max blur. Reduces average sample count at full-res for scenes with mixed focus depth.

**Current (lines 198-229):**
```hlsl
int Rings = LucentDOF.RingCount;
// ... loop uses Rings
```

**Target insertion after line 198:**
```hlsl
// Adaptive ring count: fewer rings for small CoC, saves samples at full-res.
// At CoC=1px: 1 ring (7 samples). At CoC=MaxBlurRadius: full RingCount.
// Clamp minimum to 1 ring so even the smallest blur has some samples.
int Rings = max(1, (int)round(
    float(LucentDOF.RingCount) * saturate(CenterCoC / max(LucentDOF.MaxBlurRadius * 0.5, 1.0))
));
```

**Note:** The `Rings` variable is already `int` and the loop already uses it. No other changes needed.
**Verify:** `LucentDOF.MaxBlurRadius` at this point is already divided by `GatherScale` (line 1780 in ViewExtension.cpp), so the adaptive calculation is in gather-res pixel units, which is correct.

---

## Files Changed Summary

| File | Change | Priority |
|------|--------|----------|
| `Plugins/Lucent/Shaders/LucentDOFPad.usf` | 4-tap Karis downsample | 1 — Highest |
| `Plugins/Lucent/Source/Lucent/LucentCVars.cpp` | Add GatherScale=3 text | 2 |
| `Plugins/Lucent/Source/Lucent/LucentViewExtension.h` | RenderDOFPad signature `int32→float` | 2 |
| `Plugins/Lucent/Source/Lucent/LucentViewExtension.cpp` | GatherRes 2/3 calculation, float GatherScale propagation | 2 |
| `Plugins/Lucent/Shaders/LucentDOFTemporal.usf` | AABB neighbourhood clamp | 3 |
| `Plugins/Lucent/Shaders/LucentDOF.usf` | Adaptive ring count | 4 |

---

## Confirming Current State Before Changes

### LucentDOFPad.usf — single bilinear tap (CONFIRMED needs change)
```
Line 46: OutputTexture[PixelPos] = SceneColor.SampleLevel(InputSampler, SourceUV, 0);
```

### LucentViewExtension.cpp — integer-only GatherScale (CONFIRMED needs change)
```
Line 652: static const int32 GScaleTable[] = {1, 2, 4}; NewData.DOFGatherScale = GScaleTable[...clamp 0,2...];
Line 1776: const int32 GatherScale = FMath::Max(FD.DOFGatherScale, 1);
Line 1784: const FIntPoint GatherRes = FIntPoint(ViewportSize.X / GatherScale, ViewportSize.Y / GatherScale);
```

### LucentDOFTemporal.usf — luma-diff blend (CONFIRMED can be improved)
```
Lines 70-87: LumaDiff-biased lerp(BlendMin, BlendMax, ...) — no neighbourhood clamping
```

### LucentDOF.usf — fixed ring count (CONFIRMED can be improved)
```
Line 198: int Rings = LucentDOF.RingCount;  // same for all pixels regardless of CoC
```

### LucentDOF.usf — Reinhard gather (CONFIRMED already correct — do not revert)
```
Lines 218-220: CenterTonemapped = CenterColor / (1 + Luma);  ← Reinhard, implemented last session
Lines 300-302: SampleTonemapped = SampleColor / (1 + Luma);  ← Reinhard, implemented last session
Lines 309-317: Inverse Reinhard resolve                       ← implemented last session
```

---

## Testing Notes

After implementing:
1. Test GatherScale=3 (2/3) in-engine — confirm GatherRes is ~1280×720 at 1080p
2. Compare half-res (GatherScale=1) shimmer before/after Karis pad downsample
3. Confirm sky does not get brighter at half-res after the Karis pad (sky pixels should round-trip because Karis is consistent for uniform bright input)
4. Check upsample handles float GatherScaleFloat correctly at 2/3 (non-integer DepthThreshold logic in LucentDOFUpsample.usf is fine — it uses centred UV math not a pixel grid)
