# Lucent DOF Half-Res Quality — 13-Tap Karis + Variance Clip + Pre-Gather Cap

**Date:** 2026-03-05
**Predecessor:** `docs/lucent-dof-resolution-todo.md` (all 4 changes implemented)
**Problem:** Half-res DOF still shows TAA shimmer smear despite the 4-tap Karis downsample. The current 4-tap Karis reduces a hot pixel's contribution to ~25–50%, which is insufficient — visible shimmer persists. At 2/3 res the issue is milder (1.5× block vs 2× block) but not eliminated.

---

## Root Cause Recap

The 4-tap Karis downsample is better than the old single bilinear tap, but a single hot pixel still dominates its 2×2 group. With 4 taps and Karis weighting, the hot pixel contributes `W_hot / (W_hot + 3*W_normal)` of the final value. For a 10× brighter spike: `W_hot = 1/11 ≈ 0.09`, `W_normal = 1/2 = 0.5`, so the spike contributes `0.09 / (0.09 + 1.5) ≈ 5.7%` of the weight — but its colour is 10× brighter, so it contributes `~37%` of the final colour. This diluted smear is spatially stable, so the AABB temporal clamp doesn't reject it.

**Solution:** Three complementary techniques that each attack a different stage of the pipeline.

---

## Changes (Priority Order)

---

### [1] HIGHEST — 13-Tap Partial Karis Downsample in LucentDOFPad.usf

**File:** `Plugins/Lucent/Shaders/LucentDOFPad.usf`

**What:** Replace the 4-tap Karis box filter (lines 55–86) with a 13-tap partial Karis downsample. This is the technique used by Epic's bloom downsample and many AAA DOF implementations. It uses 5 overlapping 2×2 tap groups, each independently Karis-weighted, then blends the 5 group results equally.

**Why it works:** A hot pixel appears in at most 1 of the 5 groups. Within that group, Karis attenuates it. Across the 5-group blend, the attenuated group contributes only 20%. Net result: a hot pixel contributes ~6–12% of the final colour (down from ~25–50% with the current 4-tap).

**Current code to replace (lines 55–86 — the `GatherScale > 1` branch):**
```hlsl
// 4-tap Karis-weighted box downsample.
// ...
float3 AccumColor  = float3(0.0, 0.0, 0.0);
float  AccumWeight = 0.0;

[unroll]
for (int dy = 0; dy <= 1; dy++)
{
    [unroll]
    for (int dx = 0; dx <= 1; dx++)
    {
        float2 TapViewportUV = ViewportUV + float2(dx - 0.5, dy - 0.5) * (GatherScale / ViewportSize);
        float2 TapClamped    = clamp(TapViewportUV, 0.0, 1.0);
        float2 TapSourceUV   = lerp(InputUVMin, InputUVMax, TapClamped);
        float3 TapColor      = SceneColor.SampleLevel(InputSampler, TapSourceUV, 0).rgb;

        float Luma = dot(TapColor, float3(0.2126, 0.7152, 0.0722));
        float W    = 1.0 / (1.0 + Luma);

        AccumColor  += TapColor * W;
        AccumWeight += W;
    }
}

OutputTexture[PixelPos] = float4(AccumColor / max(AccumWeight, 1e-6), 1.0);
```

**Target code:**
```hlsl
// 13-tap partial Karis downsample (5 overlapping 2×2 groups).
//
// Tap layout (in source texels, centred on ViewportUV):
//
//   A . B . C         A=(-1,-1)  B=(0,-1)  C=(+1,-1)
//   . . . . .
//   D . E . F         D=(-1, 0)  E=(0, 0)  F=(+1, 0)
//   . . . . .
//   G . H . I         G=(-1,+1)  H=(0,+1)  I=(+1,+1)
//
//   plus 4 half-offset taps:  J=(-0.5,-0.5)  K=(+0.5,-0.5)
//                              L=(-0.5,+0.5)  M=(+0.5,+0.5)
//
// Groups (each a 2×2 block of tap indices):
//   Group 0: A, B, D, E   (top-left)
//   Group 1: B, C, E, F   (top-right)
//   Group 2: D, E, G, H   (bottom-left)
//   Group 3: E, F, H, I   (bottom-right)
//   Group 4: J, K, L, M   (centre half-offset — overlaps all four)
//
// Each group is independently Karis-weighted, then the 5 results are
// blended: centre group gets 0.5, corner groups get 0.125 each.
// A hot pixel appears in at most 1 corner group + the centre group,
// capping its net contribution at ~6–12%.

float2 TexelStep = GatherScale / ViewportSize;

// Helper: sample + Karis-weight a single tap.
#define SAMPLE_TAP(OffX, OffY) \
    { \
        float2 TapUV = clamp(ViewportUV + float2(OffX, OffY) * TexelStep, 0.0, 1.0); \
        Tap = SceneColor.SampleLevel(InputSampler, lerp(InputUVMin, InputUVMax, TapUV), 0).rgb; \
    }

float3 Tap;

// Sample all 13 taps.
SAMPLE_TAP(-1.0, -1.0) float3 A = Tap;
SAMPLE_TAP( 0.0, -1.0) float3 B = Tap;
SAMPLE_TAP( 1.0, -1.0) float3 C = Tap;
SAMPLE_TAP(-1.0,  0.0) float3 D = Tap;
SAMPLE_TAP( 0.0,  0.0) float3 E = Tap;
SAMPLE_TAP( 1.0,  0.0) float3 F = Tap;
SAMPLE_TAP(-1.0,  1.0) float3 G = Tap;
SAMPLE_TAP( 0.0,  1.0) float3 H = Tap;
SAMPLE_TAP( 1.0,  1.0) float3 I = Tap;
SAMPLE_TAP(-0.5, -0.5) float3 J = Tap;
SAMPLE_TAP( 0.5, -0.5) float3 K = Tap;
SAMPLE_TAP(-0.5,  0.5) float3 L = Tap;
SAMPLE_TAP( 0.5,  0.5) float3 M = Tap;

#undef SAMPLE_TAP

// Per-group Karis-weighted average.
// KarisWeight() is defined in LucentCommon.ush: 1/(1+Luma).
// Group 0: top-left  (A, B, D, E)
float3 G0 = (A * KarisWeight(A) + B * KarisWeight(B) + D * KarisWeight(D) + E * KarisWeight(E))
           / (KarisWeight(A) + KarisWeight(B) + KarisWeight(D) + KarisWeight(E));
// Group 1: top-right (B, C, E, F)
float3 G1 = (B * KarisWeight(B) + C * KarisWeight(C) + E * KarisWeight(E) + F * KarisWeight(F))
           / (KarisWeight(B) + KarisWeight(C) + KarisWeight(E) + KarisWeight(F));
// Group 2: bottom-left  (D, E, G, H)
float3 G2 = (D * KarisWeight(D) + E * KarisWeight(E) + G * KarisWeight(G) + H * KarisWeight(H))
           / (KarisWeight(D) + KarisWeight(E) + KarisWeight(G) + KarisWeight(H));
// Group 3: bottom-right (E, F, H, I)
float3 G3 = (E * KarisWeight(E) + F * KarisWeight(F) + H * KarisWeight(H) + I * KarisWeight(I))
           / (KarisWeight(E) + KarisWeight(F) + KarisWeight(H) + KarisWeight(I));
// Group 4: centre half-offset (J, K, L, M)
float3 G4 = (J * KarisWeight(J) + K * KarisWeight(K) + L * KarisWeight(L) + M * KarisWeight(M))
           / (KarisWeight(J) + KarisWeight(K) + KarisWeight(L) + KarisWeight(M));

// Final blend: centre group weight 0.5, each corner group 0.125.
float3 Result = G4 * 0.5 + (G0 + G1 + G2 + G3) * 0.125;

OutputTexture[PixelPos] = float4(Result, 1.0);
```

**Header comment update (lines 11–14):** Change "4-tap" references to "13-tap partial Karis downsample (5 overlapping 2×2 groups)".

**Update comment at line 42:** Change "the 4-tap Karis filter uses it as the base" → "the 13-tap partial Karis filter uses it as the base".

**Dependency:** `KarisWeight()` is already defined in `LucentCommon.ush` (line 16–19) and is already `#include`'d at line 16 of this file. No new includes needed.

**Performance:** 13 texture taps vs 4 — adds 9 taps. At half-res the pad texture is ~540×960 for a 1080p viewport, so this is 9 extra bilinear taps per texel across ~520K texels = ~4.7M extra taps. Negligible on any modern GPU (well under 0.1ms).

**No dead code:** The old 4-tap loop block (lines 55–86) is fully replaced. No remnant variables.

---

### [2] HIGH — Variance Clipping in LucentDOFTemporal.usf

**File:** `Plugins/Lucent/Shaders/LucentDOFTemporal.usf`

**What:** Replace the raw min/max AABB neighbourhood clamp (lines 81–102) with variance clipping: compute the 3×3 mean (μ) and standard deviation (σ), then clamp history to `μ ± γσ` instead of the raw `[min, max]` bounding box.

**Why:** The raw AABB is too permissive — a single bright neighbour expands the box enough to let diluted smear through. Variance clipping produces a tighter envelope that scales with actual local contrast. The standard deviation naturally contracts around uniform regions (where smear hides), giving much tighter rejection.

**New CVar:** None needed — γ (gamma) can be hardcoded at 1.0 (standard UE temporal AA uses ~1.0–1.25). If artist tunability is desired later, it can be exposed then.

**Current code to replace (lines 81–106):**
```hlsl
// Neighbourhood AABB clamp: sample the 3×3 region around the current gather texel
// and clamp the reprojected history to the resulting colour bounding box.
// This rejects stale history that is spatially stable (smear that looks identical
// every frame) without requiring the pixel to flicker to be detected.
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

float3 ClampedHistory = clamp(History.rgb, NeighbourMin, NeighbourMax);

// Fixed blend: history has already been pre-rejected by the AABB clamp, so no
// luminance-diff tuning is needed.  BlendMax ~0.1 gives good ghosting resistance.
OutputTexture[PixelPos] = float4(lerp(ClampedHistory, Current.rgb, BlendMax), Current.a);
```

**Target code:**
```hlsl
// Variance clipping: compute 3×3 neighbourhood mean and standard deviation,
// then clamp reprojected history to mu ± gamma*sigma.  This is tighter than
// raw min/max AABB and catches spatially-stable smear that min/max misses.
// Gamma = 1.0 is the standard value used in AAA temporal AA (Salvi 2016).
float3 M1 = Current.rgb;          // sum of values
float3 M2 = Current.rgb * Current.rgb;  // sum of squared values

[unroll]
for (int ny = -1; ny <= 1; ny++)
{
    [unroll]
    for (int nx = -1; nx <= 1; nx++)
    {
        if (nx == 0 && ny == 0) continue;
        float2 NUV = GatherUV + float2(nx, ny) / GatherResSize;
        float3 N = CurrentGather.SampleLevel(BilinearSampler, NUV, 0).rgb;
        M1 += N;
        M2 += N * N;
    }
}

float3 Mean   = M1 / 9.0;
float3 StdDev = sqrt(max(M2 / 9.0 - Mean * Mean, 0.0));

// Gamma controls how many standard deviations from the mean are allowed.
// 1.0 = tight rejection (best for eliminating stable smear).
// Higher values = more permissive (less flickering on valid detail).
const float Gamma = 1.0;
float3 ClipMin = Mean - Gamma * StdDev;
float3 ClipMax = Mean + Gamma * StdDev;

float3 ClampedHistory = clamp(History.rgb, ClipMin, ClipMax);

// Fixed blend: history has already been pre-rejected by variance clipping, so no
// luminance-diff tuning is needed.  BlendMax ~0.1 gives good ghosting resistance.
OutputTexture[PixelPos] = float4(lerp(ClampedHistory, Current.rgb, BlendMax), Current.a);
```

**Header comment update (lines 6–12):** Change "3×3 colour AABB neighbourhood clamp" → "3×3 variance clipping (μ ± γσ)". Update the explanation to describe variance clipping rather than raw AABB.

**No dead code:** The `NeighbourMin/Max` variables are removed entirely. The new `M1/M2/Mean/StdDev/ClipMin/ClipMax` variables replace them.

**No C++ changes:** The `FLucentDOFTemporalCS::FParameters` struct is unchanged — same inputs, same outputs. The blend logic change is entirely shader-side.

---

### [3] MEDIUM — Pre-Gather Soft Luminance Cap in LucentDOFPad.usf Output

**File:** `Plugins/Lucent/Shaders/LucentDOFPad.usf`

**What:** After the 13-tap downsample computes `Result`, apply a soft luminance cap before writing to `OutputTexture`. This catches any residual spikes that survive the partial Karis downsample. The cap value reuses the existing `MaxBokehLuminance` CVar (already exposed in `r.Lucent.DOF.MaxBokehLuminance`, already in `FFrameData`, default 50.0).

**Why:** Even with the 13-tap partial Karis, a sufficiently bright spike (100× background) can still contribute meaningful energy. A luminance cap in the pad output provides a hard ceiling before the gather kernel spreads it.

#### 3a. Shader parameter addition — LucentDOFPad.usf

**Add new parameter declaration after line 28:**
```hlsl
float  MaxPadLuminance;  // soft luminance cap on pad output (0 = disabled)
```

**Add to both output paths:**

At the end of the `GatherScale <= 1` branch (line 51), before `return`:
```hlsl
// Soft luminance cap (reuses MaxBokehLuminance).
float4 PadColor = SceneColor.SampleLevel(InputSampler, SourceUV, 0);
if (MaxPadLuminance > 0.0)
{
    float PadLuma = dot(PadColor.rgb, float3(0.2126, 0.7152, 0.0722));
    PadColor.rgb *= min(1.0, MaxPadLuminance / max(PadLuma, 1e-6));
}
OutputTexture[PixelPos] = PadColor;
```

At the end of the 13-tap path, after computing `Result`, before writing `OutputTexture`:
```hlsl
// Soft luminance cap: clamps residual spikes that survive the partial Karis.
if (MaxPadLuminance > 0.0)
{
    float ResultLuma = dot(Result, float3(0.2126, 0.7152, 0.0722));
    Result *= min(1.0, MaxPadLuminance / max(ResultLuma, 1e-6));
}

OutputTexture[PixelPos] = float4(Result, 1.0);
```

#### 3b. C++ parameter struct — LucentViewExtension.cpp

**File:** `Plugins/Lucent/Source/Lucent/LucentViewExtension.cpp`

In `FLucentDOFPadCS::FParameters` (line 263–273), add after `GatherScale`:
```cpp
SHADER_PARAMETER(float, MaxPadLuminance)
```

#### 3c. C++ parameter assignment — RenderDOFPad()

**File:** `Plugins/Lucent/Source/Lucent/LucentViewExtension.cpp`

In `RenderDOFPad()` (around line 1556), after `PassParams->GatherScale = GatherScale;`, add:
```cpp
PassParams->MaxPadLuminance = RenderThread_FrameData.DOFMaxBokehLuminance;
```

This reuses the existing `DOFMaxBokehLuminance` field — no new CVar, no new FFrameData field, no new FLucentSettings field. The artist's existing MaxBokehLuminance slider now does double duty: caps both the pad output AND the post-gather inverse-Reinhard resolve.

**No dead code:** This is purely additive — no existing code is removed.

---

## Files Changed Summary

| File | Change | Priority |
|------|--------|----------|
| `Plugins/Lucent/Shaders/LucentDOFPad.usf` | Replace 4-tap Karis with 13-tap partial Karis; add soft luminance cap | 1, 3 |
| `Plugins/Lucent/Shaders/LucentDOFTemporal.usf` | Replace AABB clamp with variance clipping (μ ± γσ) | 2 |
| `Plugins/Lucent/Source/Lucent/LucentViewExtension.cpp` | Add `MaxPadLuminance` to `FLucentDOFPadCS::FParameters`; assign in `RenderDOFPad()` | 3 |

**Files NOT changed:**
- `LucentCVars.h/.cpp` — no new CVars (reuses `DOFMaxBokehLuminance`)
- `LucentViewExtension.h` — no signature changes
- `LucentSettings.h/.cpp` — no new settings fields
- `LucentPanel.cpp` — no new panel controls
- `LucentDOF.usf` — gather shader is unchanged
- `LucentCommon.ush` — `KarisWeight()` already exists, no changes needed

---

## Dead Code Audit

Each change fully replaces its predecessor — no orphaned variables or unreachable branches:

1. **LucentDOFPad.usf:** The `AccumColor/AccumWeight` loop (lines 64–84) and surrounding comments (lines 55–63) are entirely replaced by the 13-tap code. The `Tap` temporary and `SAMPLE_TAP` macro are `#undef`'d after use.

2. **LucentDOFTemporal.usf:** The `NeighbourMin/NeighbourMax` variables (lines 85–86) and the min/max loop (lines 88–100) are entirely replaced by the `M1/M2` variance code. The `ClampedHistory` variable is reused with the new clip bounds.

3. **LucentViewExtension.cpp:** The `MaxPadLuminance` parameter is purely additive in both the struct and the assignment. No existing parameters are removed.

---

## Testing Notes

After implementing all 3 changes:

1. **Half-res shimmer test:** Place a metallic sphere in direct sunlight (high specular). Set `r.Lucent.DOF.GatherScale 1` (half-res). Defocus and verify shimmer is suppressed vs the 4-tap baseline.

2. **Sky brightness regression:** Check that uniform bright sky does not get dimmer at half-res after the luminance cap. At `MaxBokehLuminance=50` and typical sky luminance ~1.5, the cap should not activate for sky pixels.

3. **Temporal stability:** Pan the camera slowly and verify no flickering or excessive ghosting from the variance clipping. If flickering occurs, increase Gamma from 1.0 to 1.25 in the shader.

4. **2/3 res test:** Verify the 13-tap works correctly at non-integer GatherScale (1.5). The `TexelStep` calculation should handle this naturally.

5. **Performance:** Profile `Lucent.DOFPad` pass time at half-res 1080p. Expected increase: <0.05ms from the additional 9 texture taps. The variance clipping in temporal adds zero extra taps (same 9 neighbour samples, just different math).
