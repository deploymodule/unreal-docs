# Screen Space Fog Scattering — Plugin Analysis & Improvement Reference

**Plugin:** Screen Space Fog Scattering v1.31
**Author:** Dmitry Karpukhin
**Source Path:** `C:\Program Files\Epic Games\UE_5.7\Engine\Plugins\Marketplace\ScreenSpf1162ed1f13cV9`
**UE Compatibility:** 5.3 – 5.7
**Category:** Rendering
**Purpose of this doc:** Full technical breakdown of the plugin's architecture, its current strengths and limitations, every quality and performance improvement identified, and a reference for building an improved version.

---

## 1. What the Plugin Does

Enhances Unreal Engine's built-in Exponential Height Fog by simulating **light scattering** through the fog medium. Without this plugin, UE's height fog is a flat overlay that doesn't react to light sources or produce the soft, glowing haze you see in real foggy environments.

The core technique: extract the fog density per-pixel from the scene depth, apply a **multi-pass blur** (inspired by the COD: Advanced Warfare bloom pipeline) to a fog-tinted version of the scene color, then composite the blurred result back into the scene. The blur simulates how scattered light spreads outward through the fog medium.

**Visual result:** Bright objects (lights, sky, emissive surfaces) appear to "glow" softly into surrounding fog. Dense fog regions fill in with a soft, luminous haze instead of a flat color.

---

## 2. Architecture Overview

### The 4-Phase Render Pipeline

All passes use **compute shaders** (no pixel shaders, no render targets with draw calls). The effect hooks into `PrePostProcessPass_RenderThread`, which runs **before Depth of Field** — this is the right place for fog since DOF should blur an already-fogged image.

```
┌─────────────────────────────────────────────────────────────────┐
│  PASS 1: SETUP  (full resolution compute)                        │
│                                                                  │
│  Input:  SceneColor, SceneDepth                                  │
│  Output: SetupTexture                                            │
│    RGB = SceneColor * tonemap(fogDensity)                        │
│    A   = tonemap(fogDensity)                                     │
│                                                                  │
│  Work done per pixel:                                            │
│    - Reconstruct world position from depth                       │
│    - CalculateHeightFog() from HeightFogCommon.ush               │
│    - Combine Volumetric Fog (3D LUT lookup)                      │
│    - Combine Sky Atmosphere (aerial perspective LUT)             │
│    - Combine Heterogeneous Volumes (texture sample)              │
│    - Combine Volumetric Clouds (texture sample)                  │
└─────────────────────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────────┐
│  PASS 2: DOWNSAMPLE CHAIN  (N passes, default 8)                 │
│                                                                  │
│  Each pass halves resolution:                                    │
│  1920×1080 → 960×540 → 480×270 → 240×135 → ...                 │
│                                                                  │
│  Pass 0: Uses SetupTexture directly (no shader, free)            │
│  Pass 1+: Downsample compute shader with:                        │
│    - 4-tap  (Quality 0 / Low)                                    │
│    - 8-tap  (Quality 1-2 / Medium-High)                          │
│    - 13-tap (Quality 3 / Epic) — COD:AW style dual-Kawase        │
│    + Karis average luminance weighting (anti-firefly)            │
└─────────────────────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────────┐
│  PASS 3: UPSAMPLE + COMBINE CHAIN  (N-1 passes)                  │
│                                                                  │
│  Each pass doubles resolution and blends with same-level mip:    │
│    - 5-tap  (Quality 0-1 / Low-Medium)                           │
│    - 9-tap  (Quality 2-3 / High-Epic) — tent filter              │
│                                                                  │
│  Blend: lerp(currentMip, upsampledFromBelow, Radius)             │
│  Radius (r.SSFS.Radius) controls how "spread out" the effect is  │
└─────────────────────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────────┐
│  PASS 4: RECOMBINE  (full resolution compute)                    │
│                                                                  │
│  lerp(SceneColor, ScatteredResult, fogDensity)                   │
│  Writes back to a temp texture → AddCopyTexturePass to SceneColor│
└─────────────────────────────────────────────────────────────────┘
```

### Why Downsample + Upsample?

Blurring at full resolution is expensive. Instead, the chain progressively halves the resolution (cheap reads, small outputs), then re-expands (combining each level). This produces a wide, smooth blur from a relatively small number of samples — the same reason Unreal uses it for Bloom. The number of passes controls the maximum blur radius.

**Reference:** Jorge Jimenez, "Next Generation Post Processing in Call of Duty: Advanced Warfare," SIGGRAPH 2014. Also see: https://www.froyok.fr/blog/2021-12-ue4-custom-bloom/ (UE4 implementation breakdown).

---

## 3. C++ Structure

### File Map

| File | Role |
|------|------|
| `SSFS.h / SSFS.cpp` | Module class. Registers the `Shaders/` directory with the engine at startup so `.usf` files are found. |
| `SSFSSubsystem.h / SSFSSubsystem.cpp` | `UEngineSubsystem` — creates the `FScreenSpaceFogScatteringViewExtension` on game start, tears it down cleanly on shutdown. The subsystem is the "owner" that keeps the view extension alive. |
| `SSFSViewExtension.h / SSFSViewExtension.cpp` | Core rendering logic. Inherits `FSceneViewExtensionBase`. Implements `PrePostProcessPass_RenderThread` to inject the effect, and `IsActiveThisFrame_Internal` for enable/disable logic. Contains all 4 pass functions. |
| `SSFSHeightFog.h` | Contains `SetupFogUniformParametersSSFS()` — creates a fog uniform buffer by mirroring the engine's private `FFogUniformParameters` struct. This is required because `FogRendering.h` is private to the Renderer module. |

### The Fog Impostor Uniform Buffer

The most interesting architectural trick in the plugin. UE's fog shaders (`HeightFogCommon.ush`) expect a specific uniform buffer named `FogStruct`. But that buffer type lives in `FogRendering.h`, which is private to the Renderer module.

The solution: define a near-identical struct `FFogUniformParametersSSFS`, fill it from the same `FViewInfo` fields that UE's internal code uses, then alias it:

```cpp
// In SSFSSetup.usf:
#include "/Engine/Generated/UniformBuffers/FogStructSSFS.ush"
#define FogStruct FogStructSSFS  // Trick HeightFogCommon.ush into using our impostor
#include "/Engine/Private/HeightFogCommon.ush"
```

This lets the plugin reuse UE's exact fog calculation functions without touching private code. Fragile (breaks if Epic changes the fog UB layout) but clever. Version-check macros handle UE 5.3/5.4/5.5/5.6+ layout differences.

### Shader Permutation Strategy

The Setup shader has **9 boolean permutations** — one per fog feature (height fog, start distance, inscattering texture, second fog term, directional inscattering, volumetric fog, aerial perspective, heterogeneous volumes, volumetric clouds).

`RemapPermutation()` prunes impossible combos: if height fog is off, all height-fog sub-features are forced false. This reduces the compiled variant count significantly.

At runtime, the C++ checks which features are actually active in the scene and selects the right permutation — zero dead code paths execute on the GPU.

### Console Variables

| CVar | Default | Notes |
|------|---------|-------|
| `r.SSFS` | 1 | Master enable/disable |
| `r.SSFS.QualityPreset` | 2 | 0=Low, 1=Medium, 2=High, 3=Epic |
| `r.SSFS.PassAmount` | 8 | Downsample/upsample depth, max 12 |
| `r.SSFS.Radius` | 0.9 | Upsample blend ratio, 0.8-0.99 recommended |
| `r.SSFS.Intensity` | 1.0 | Overall effect multiplier |
| `r.SSFS.Format` | 1 | 0=R11G11B10, 1=FloatRGBA |
| `r.SSFS.LuminanceWeighting` | 1 | Karis anti-firefly filter |
| `r.SSFS.SkyAtmosphere` | 0 | Include sky atmosphere contribution |
| `r.SSFS.SkyAtmosphereIntensity` | 1.0 | Sky atmosphere strength |
| `r.SSFS.VolumetricCloud` | 0 | Include volumetric cloud (experimental) |
| `r.SSFS.VolumetricCloudIntensity` | 1.0 | Cloud contribution strength |
| `r.SSFS.VolumetricFogILSIntensity` | 1.0 | Volumetric fog integrated light scattering |
| `r.SSFS.HeterogeneousVolumes` | 1 | Include heterogeneous volumes |

---

## 4. Shader Structure

### `SSFSCommon.ush`
Shared header included by all shaders. Declares the viewport macro (`SCREEN_PASS_TEXTURE_VIEWPORT`), texture/sampler inputs, and the tonemap utility functions:
```hlsl
float SimpleTonemap(float Value)        { return pow(Value, 1.0 / 2.2); }
float InverseSimpleTonemap(float Value) { return pow(Value, 2.2); }
```
These are used to compress the fog density into a modified range before the blur chain so that bright values don't blow out during downsampling.

### `SSFSSetup.usf`
The most complex shader. Per-pixel operations:
1. Sample SceneColor (bilinear) and SceneDepth
2. Reconstruct world position via `SvPositionToTranslatedWorld()`
3. Call `CalculateHeightFog(WorldPositionRelativeToCamera)` — this is UE's full height fog function including exponential falloff, max opacity, color, second fog term, directional inscattering, and inscattering cubemap
4. Optionally: look up volumetric fog from the 3D integrated light scattering texture
5. Optionally: sample sky atmosphere aerial perspective LUT
6. Optionally: sample heterogeneous volume radiance texture
7. Optionally: sample volumetric cloud texture
8. Combine all contributions, write RGB=`SceneColor * tonemap(density)`, A=`tonemap(density)`

### `SSFSDownsample.usf`
Samples from the previous mip using a kernel of 4/8/13 taps depending on quality preset. The **13-tap Epic kernel** is the COD:AW dual-Kawase pattern — good frequency separation with fewer samples than a full Gaussian. Optional Karis luminance weighting prevents bright fireflies from amplifying through the chain.

### `SSFSUpsampleCombine.usf`
Reads two textures: the current mip level and the previous (lower-res) upsample result. Blends them: `lerp(current, upsampledBelow, Radius)`. Uses a 5-tap or 9-tap (3×3 Gaussian) filter for smoothing. The `Radius` parameter (0.8-0.99) controls how much the lower-frequency (larger blur) contributes — higher radius = more spread, more glow.

### `SSFSRecombine.usf`
Final composite. Reads original SceneColor, the fully upsampled scattering result, and the fog density from the SetupTexture alpha. Blends:
```hlsl
float3 OutColor = lerp(SceneColor.rgb, ScatteringColor, InverseSimpleTonemap(HeightFog));
```
Writes back to a full-res output texture which is then copied to SceneColor.

---

## 5. What It Does Well

- **All compute shaders** — no legacy pixel shader rendering path. Better for modern GPU scheduling.
- **Karis average anti-firefly** — production-quality filter, the same technique Unreal uses in its own bloom.
- **Permutation-based dead code elimination** — inactive features compile out entirely, no runtime branching.
- **Fog impostor trick** — reuses UE's own fog math without duplicating it. Stays accurate across UE versions.
- **Pre-DOF injection** — runs before depth of field, so DOF correctly blurs an already-scattered scene.
- **Stereo rendering support** — uses `AddCopyTexturePass` with explicit viewport rect handling for VR compatibility.
- **Multiple fog source support** — height fog, volumetric fog, sky atmosphere, heterogeneous volumes, clouds all composited correctly.
- **Version compatibility** — extensive `#if UE_VERSION_NEWER_THAN_OR_EQUAL` guards for 5.3 through 5.7.

---

## 6. Limitations & Issues

### Visual / Quality Limitations

| Issue | Description |
|-------|-------------|
| **No depth-aware blur** | The downsample/upsample chain is a pure image-space blur with no depth knowledge. Fog scattering bleeds across hard depth edges — a sharp foreground object silhouetted against foggy background will have its edges "bleed" the fog glow. A bilateral filter that rejects samples across large depth discontinuities would fix this. |
| **No directional phase function** | Real light scattering (Mie and Rayleigh) is strongly directional. Mie scattering (fog, clouds) produces a forward-heavy lobe — the sky near the sun should scatter dramatically more than away from it. This plugin blurs uniformly in all directions. |
| **Simple tonemap** | `pow(x, 1/2.2)` is a basic gamma curve, not a proper tone mapping operator. In scenes with extreme HDR values it can produce incorrect colors. Could use Reinhard, ACES, or Khronos PBR Neutral. |
| **No temporal stability** | No temporal accumulation, no jitter. Fast camera movement can cause shimmering/flickering in the scattering result, especially on high-frequency fog density gradients. |
| **No chromatic scattering** | Real scattering is wavelength-dependent (Rayleigh: shorter wavelengths scatter more → blue sky). Uniform RGB blurring doesn't capture this. |

### Usability Limitations

| Issue | Description |
|-------|-------------|
| **CVar-only configuration** | No `UObject`, no `UActorComponent`, no Post Process Volume integration. Designers can't vary settings spatially or per-sequence. Everything goes through the console. |
| **No Blueprint exposure** | Can't be driven at runtime from Blueprint logic (e.g., ramping fog scatter during a storm). |
| **No editor visualization** | No way to preview the fog density mask in the editor to understand what the shader "sees." |

### Code Quality Issues

| Issue | Location | Description |
|-------|----------|-------------|
| **Redundant `pow()` roundtrip** | `SSFSSetup.usf:155`, `SSFSRecombine.usf:19` | Alpha stored as `pow(x, 1/2.2)`, read back as `pow(that, 2.2)` = identity. Net effect: two `pow()` calls per pixel for nothing. |
| **Member array state** | `SSFSViewExtension.h:35-36` | `DownsampleMipMaps` and `UpsampleMipMaps` are instance member variables cleared each frame, not local per-call. Can cause issues with multi-view rendering. |
| **`UpsampleMipMaps.Append()`** | `SSFSViewExtension.cpp:202` | Copies the entire downsample array every frame to use as starting data for upsample. Wasteful heap allocation. |
| **FString allocations in render loop** | `SSFSViewExtension.cpp:169-177, 208-215` | String concatenations + heap allocations inside a per-frame, per-pass loop on the render thread. Should be pre-built or compile-gated. |
| **Static CVar lookup** | `SSFSViewExtension.cpp:237-238` | `IsVolumetricCloudRenderTargetValid()` uses `static` local `IConsoleVariable*` pointers — won't update if CVars are removed/added at runtime. |
| **Setup texture format hardcoded** | `SSFSViewExtension.cpp:255` | Setup texture always `PF_FloatRGBA` (64-bit) regardless of `r.SSFS.Format` setting. Inconsistent with downsample/upsample which respect the CVar. |
| **No compute shader bounds check** | All `.usf` files | No `if (DispatchThreadId >= OutputSize) return;` guard. At non-power-of-2 mip sizes, extra threads run and write to out-of-bounds UAV positions. |

---

## 7. Performance Analysis

### Estimated Cost at 1080p (RTX 3070 class GPU)

| Pass | Approx Cost | Notes |
|------|-------------|-------|
| Setup (full res) | ~0.4–0.8ms | Heaviest per-pixel work: exp(), 3D LUT, matrix multiply |
| Downsample chain (7 passes) | ~0.1–0.15ms | Rapidly shrinking textures, cheap |
| Upsample chain (7 passes) | ~0.1–0.20ms | Slightly more work per tap |
| Recombine (full res) | ~0.05–0.10ms | 3 texture reads + lerp |
| AddCopyTexturePass | ~0.05–0.10ms | Full-res texture copy |
| **Total** | **~0.7–1.35ms** | Scales ~4x to 4K |

The Setup pass is the dominant cost — it scales quadratically with resolution and does the most ALU work per pixel.

### Performance Improvements & Estimated Gains

All gains estimated at 1080p on RTX 3070 class hardware. Real numbers require Unreal Insights / RenderDoc profiling.

---

#### Perf Fix 1: Half-Resolution Setup Pass
**Estimated saving: ~0.30–0.50ms at 1080p, ~1.5ms at 4K**

The Setup pass runs at full resolution doing expensive per-pixel work (exp(), matrix multiply, multiple texture samples). Fog density is spatially low-frequency — it varies smoothly across the screen because it's derived from depth, which changes gradually. Running Setup at half resolution is nearly imperceptible visually.

**Implementation:** Change `Desc.Extent = SceneColor.ViewRect.Size()` to `SceneColor.ViewRect.Size() / 2` in the Setup function. Adjust UV calculation to account for the reduced extent. The first real downsample pass naturally handles the upscale.

**Resolution scaling:**

| Resolution | Current Setup | Half-Res Setup | Saving |
|------------|--------------|----------------|--------|
| 1080p | ~0.50ms | ~0.15ms | ~0.35ms |
| 1440p | ~0.90ms | ~0.25ms | ~0.65ms |
| 4K | ~2.00ms | ~0.50ms | ~1.50ms |

---

#### Perf Fix 2: Remove Redundant `pow()` Roundtrip on Alpha
**Estimated saving: ~0.01ms (negligible but free)**

In `SSFSSetup.usf`: `Output.a = SimpleTonemap(FinalFog.a)` → stores `pow(density, 1/2.2)`
In `SSFSRecombine.usf`: `InverseSimpleTonemap(HeightFog)` → computes `pow(pow(density, 1/2.2), 2.2)` = `density`

These cancel to identity. Store the raw fog density in alpha, read it raw in Recombine. Two fewer `pow()` calls per pixel at full resolution.

```hlsl
// SSFSSetup.usf — change:
Output[DispatchThreadId] = float4(SceneColor * SimpleTonemap(FinalFog.a), FinalFog.a); // raw alpha

// SSFSRecombine.usf — change:
float HeightFog = Texture2DSample(SetupTexture, InputSampler, UV).a; // already raw
float3 OutColor = lerp(SceneColor.rgb, ScatteringColor, HeightFog);  // no InverseSimpleTonemap
```

---

#### Perf Fix 3: Setup Texture Format
**Estimated saving: ~0.03–0.06ms**

Setup texture is always `PF_FloatRGBA` (64 bits/pixel). The downsample/upsample textures correctly respect `r.SSFS.Format` and can use `PF_FloatR11G11B10` (32 bits/pixel). The Setup texture contains fog-tinted scene color — 11:11:10 precision is sufficient for this intermediate result.

The Setup texture is written once and read twice (first downsample pass + Recombine) = 3 bandwidth passes × 16MB at 1080p = 48MB. At 400GB/s bandwidth, switching to R11G11B10 saves ~24MB = ~0.06ms bandwidth.

```cpp
// SSFSViewExtension.cpp Setup() — change:
Desc.Format = CVarSSFSFormat.GetValueOnRenderThread() ? PF_FloatRGBA : PF_FloatR11G11B10;
// Consistent with downsample/upsample passes
```

---

#### Perf Fix 4: Eliminate FString Allocations in Render Loop
**Estimated saving: ~0.02–0.05ms render thread CPU**

16 string heap allocations per frame inside the downsample and upsample loops. These happen on the render thread every frame.

```cpp
// Current (per-frame, per-pass string construction):
const FString PassName = "Downsample 1/" + FString::FromInt(Divider) + " (" + ...;
const FString* TextureName = GraphBuilder.AllocObject<FString>("SSFS.Downsample(1/" + ...);
```

**Fix:** Pre-compute and cache pass name arrays at construction time, or gate behind `#if WANTS_DRAW_MESH_EVENTS` so they compile out entirely in Shipping builds (which is where performance matters most). The `RDG_EVENT_NAME` macro already does this for GPU stats — consistency would also help here.

---

#### Perf Fix 5: Eliminate Array Copy Every Frame
**Estimated saving: ~0.001ms (code quality)**

`UpsampleMipMaps.Append(DownsampleMipMaps)` copies 8–12 `FScreenPassTexture` structs (~200 bytes) plus a heap allocation every frame. The loop then overwrites every entry. Instead, pass `DownsampleMipMaps` directly to the upsample loop indexed in reverse, using a separate small `TArray` only for the outputs.

---

#### Perf Fix 6: Skip `AddCopyTexturePass` for Mono Rendering
**Estimated saving: ~0.05–0.10ms**

The plugin writes Recombine output to a temp texture then copies it back to SceneColor. The copy exists for stereo rendering correctness. For non-stereo (mono) views this is an unnecessary full-resolution texture copy (~16MB at 1080p).

```cpp
// In PrePostProcessPass_RenderThread:
if (View.StereoPass != EStereoscopicPass::eSSP_FULL)
{
    // Mono: can write directly, but still need the copy for correct viewport handling
    // OR: check if Recombine can write directly to SceneColor when ViewRect matches full extent
}
```

---

#### Perf Fix 7: Compute Shader Bounds Checks
**Estimated saving: ~0.0ms (correctness fix)**

At lower mip levels, texture dimensions are not multiples of 8, so extra thread groups are dispatched. These threads run and write to UAV positions outside the valid texture region. Hardware accepts these writes silently on most platforms but it's technically undefined behavior and wastes work.

Add at the top of each compute shader entry point:
```hlsl
[numthreads(GROUP_SIZE, GROUP_SIZE, 1)]
void DownsampleCS(uint2 DispatchThreadId : SV_DispatchThreadID)
{
    if (any(DispatchThreadId >= uint2(OutputWidth, OutputHeight))) return; // bounds check
    ...
}
```

---

### Combined Performance Summary

| Fix | GPU Saving | CPU (RT) Saving | Complexity |
|-----|-----------|-----------------|------------|
| Half-res Setup | ~0.35ms (1080p) / ~1.5ms (4K) | — | Medium |
| Setup texture format | ~0.04ms | — | Low |
| Skip mono copy | ~0.07ms | — | Low |
| pow() roundtrip | ~0.01ms | — | Trivial |
| FString allocs | — | ~0.03ms | Low |
| Array copy | — | ~0.001ms | Low |
| Bounds check | ~0.0ms | — | Low |
| **Total** | **~0.47–0.62ms** | **~0.03ms** | |

At 1080p you get roughly **~0.5ms total savings** — about a 35–50% cost reduction on this effect. At 4K the savings are ~2ms, making this effect go from potentially frame-rate-impacting to genuinely cheap.

---

## 8. Quality Improvement Opportunities

These go beyond bug fixes into genuinely new features.

### Quality Fix 1: Depth-Aware Bilateral Blur

**Problem:** The current downsample/upsample chain is a pure image-space blur. It has no knowledge of depth, so fog scattering bleeds across sharp silhouettes. A rock in the foreground with fog behind it will have its edges softened by the scattered fog glow.

**Solution:** In the upsample combine pass, reject samples from the previous mip that have very different depth from the current pixel. This is called a **bilateral filter** or **joint bilateral upsample** (also used in UE's SSAO and GTAO for upsampling).

Requires passing the depth texture to the upsample shader and comparing depths during the multi-tap gather.

```hlsl
// In UpsampleCombine shader — weighted by depth similarity:
float depthCenter = SampleDepth(UV);
float depthWeight = exp(-abs(depthSample - depthCenter) * DepthSensitivity);
Weight += Weights[i] * depthWeight;
Color  += Texture2DSampleLevel(Texture, Sampler, UV + offset, 0).rgb * Weights[i] * depthWeight;
```

---

### Quality Fix 2: Directional Phase Function (Mie Scattering)

**Problem:** The blur is isotropic — it scatters equally in all directions. Real fog/haze (Mie scattering) has a strong **forward lobe** — light scatters most toward the viewer when looking toward the light source. Standing with your back to the sun in fog, the fog looks dark. Facing the sun, it glows brightly.

**Solution:** In the Setup pass, compute a directional scattering weight based on the angle between the view direction and the primary directional light direction. Use a Henyey-Greenstein phase function:

```hlsl
float HenyeyGreenstein(float cosTheta, float g)
{
    float g2 = g * g;
    return (1.0 - g2) / (4.0 * PI * pow(1.0 + g2 - 2.0 * g * cosTheta, 1.5));
}
// g = 0.0 (isotropic) to 0.9 (strong forward scatter)
// Multiply fog density by the phase function result before storing to SetupTexture
```

This single addition dramatically improves visual realism at the cost of one dot product and one `pow()` per pixel in Setup.

---

### Quality Fix 3: Proper HDR-Aware Tonemapping

**Problem:** `pow(x, 1/2.2)` is a basic gamma curve. It compresses bright values but not accurately for HDR content — it clips differently than display tonemappers do.

**Solution:** Use a proper tonemapping operator in the pre-tonemap step. Good options for intermediate use (must be invertible):

```hlsl
// Reinhard (invertible):
float Reinhard(float v)        { return v / (1.0 + v); }
float InvReinhard(float v)     { return v / (1.0 - v); }

// Reinhard extended (handles peak luminance):
float ReinhardEx(float v, float peak)  { return v * (1.0 + v / (peak * peak)) / (1.0 + v); }
```

Reinhard is more HDR-appropriate than gamma and still fully invertible.

---

### Quality Fix 4: Temporal Accumulation

**Problem:** Without temporal information, the effect can shimmer on camera movement, particularly when fog gradients are steep or the scene has many fine details.

**Solution:** Blend with the previous frame's result using reprojection. This is the same technique used by TAA, SSAO, and GTAO:

1. In Recombine (or a new pass after it), reproject the previous frame's scattering result using the velocity buffer
2. Blend: `output = lerp(currentFrame, reprojectedPrevious, TemporalBlendFactor)` (typically 0.05–0.2)
3. Detect disocclusion (pixels with no valid previous frame data) and fall back to current frame

Requires storing one additional render target between frames (the previous frame's scattering result) and access to the motion vector buffer.

---

### Quality Fix 5: Post Process Volume Integration

**Problem:** Currently all settings are global CVars. You can't have different fog scatter intensity in a cave vs. outdoors, or animate settings per-sequence.

**Solution:** Add a `UPostProcessComponent` or register custom settings with the Post Process Volume system. UE provides `FPostProcessSettings` with `UPROPERTY` support:

```cpp
USTRUCT(BlueprintType)
struct FSSFSSettings
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(ClampMin="0.0", ClampMax="4.0"))
    float Intensity = 1.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(ClampMin="0.0", ClampMax="12"))
    int32 PassAmount = 8;

    // etc.
};
```

Then in the view extension, read from `View.FinalPostProcessSettings` (blended by the engine across volumes) rather than raw CVars.

---

### Quality Fix 6: Blueprint-Accessible Runtime Control

Add a `USceneComponent` or `UActorComponent` subclass with Blueprint-exposed functions:

```cpp
UFUNCTION(BlueprintCallable, Category="SSFS")
void SetScatterIntensity(float NewIntensity);

UFUNCTION(BlueprintCallable, Category="SSFS")
void SetScatterRadius(float NewRadius);

UFUNCTION(BlueprintCallable, Category="SSFS")
void FadeToSettings(FSSFSSettings Target, float Duration);
```

This enables designers to control the effect through cutscene sequences, weather systems, time-of-day transitions, or gameplay events.

---

## 9. Reference Technique: The Dual-Kawase Bloom Pipeline

The downsample/upsample approach used here is directly derived from this well-documented technique. Understanding it deeply is essential before building an improved version.

**Key papers and posts to read:**
- Jimenez, "Next Generation Post Processing in COD:AW," SIGGRAPH 2014 — the original source
- https://www.froyok.fr/blog/2021-12-ue4-custom-bloom/ — UE4 Bloom implementation walkthrough, directly applicable
- Kawase, "Frame Buffer Postprocessing Effects in DOUBLE-S.T.E.A.L" — original dual-Kawase filter paper

**Core insight:** Progressively downsampling and upsampling creates a "pyramid" of blurred images. Each level of the pyramid represents a different frequency of the original image (low mips = large blur = low frequency). Recombining them sums frequencies back together, producing a wide smooth blur with good frequency characteristics from a small number of samples.

---

## 10. Building Your Own Improved Version — Starting Checklist

When you come back to create your own enhanced fog scattering plugin, here's what to set up first:

**C++ infrastructure (same as this plugin):**
- [ ] Module that registers shader directory
- [ ] `UEngineSubsystem` to own the view extension lifetime
- [ ] `FSceneViewExtensionBase` subclass, hook `PrePostProcessPass_RenderThread`
- [ ] Fog impostor UB (copy `SSFSHeightFog.h` as starting point, update for target UE version)

**Shader infrastructure:**
- [ ] Common header with viewport setup and tonemap utilities
- [ ] Setup compute shader (derive from the existing `SSFSSetup.usf` — the fog math is solid)
- [ ] Downsample compute shader (reuse exactly — the Kawase filter is correct)
- [ ] Upsample compute shader (start from existing, add bilateral depth weighting)
- [ ] Recombine compute shader (simplified version without redundant tonemap on alpha)

**First improvements to add (in order of impact):**
1. Half-resolution Setup pass
2. Depth-aware bilateral upsample
3. Henyey-Greenstein phase function in Setup
4. Post Process Volume integration for designer control
5. Temporal accumulation (most complex, do last)

**Files from this plugin to keep unchanged:**
- `SSFSDownsample.usf` — the Kawase filter implementation is correct and well-optimized
- The permutation system structure in Setup — sound architecture, just add new permutations as needed
- The `UEngineSubsystem` approach — correct lifetime management

**Files to significantly rewrite:**
- `SSFSSetup.usf` — add phase function, fix alpha storage, run at half-res
- `SSFSUpsampleCombine.usf` — add depth-aware bilateral filtering
- `SSFSViewExtension.cpp` — fix member array issues, FString allocs, add volume integration
