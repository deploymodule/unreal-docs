# Aether Kuwahara Filter — Remaining Work

## Status Summary

The 5-pass compute pipeline IS functional:
- Tensor (Pass 1), ZBlur (Pass 3), ChromaDepth (Pass 4), Recombine (Pass 5) all produce visible output
- Kuwahara (Pass 2) runs correctly with the zero-init fix applied in this session
- Preset registration warning fixed (DefaultGame.ini updated)
- Architecture confirmed valid: SceneViewExtension + compute shaders + AddCopyTexturePass works

Two major gaps remain vs the reference oil-painting look:
1. The pipeline runs **pre-tonemapping on HDR data** — should run post-tonemapping on LDR
2. No **edge/outline detection pass** — the reference has dark outlines drawn over Kuwahara patches

---

## Phase 1: Move Kuwahara Pipeline to Post-Tonemapping

### Problem

`PrePostProcessPass_RenderThread` runs before ALL post-processing (DOF, Bloom, Tonemapping). Scene color at this point is **linear HDR** with values often >1.0 (specular highlights, sky, emissives). Running Kuwahara on HDR data causes:

- Specular highlights dominate variance calculations — a sector spanning a bright window vs shadow has enormous HDR variance, even if visually (post-tonemap) the difference is subtle
- The sector soft-minimum is biased by HDR range, not perceptual appearance
- Result: looks like a blur rather than crisp oil-painting patches

Material-based Kuwahara implementations (like the reference) run **after tonemapping** on LDR [0,1] data, where variance maps directly to what the viewer sees.

### Solution

Switch from `PrePostProcessPass_RenderThread` to `SubscribeToPostProcessingPass` hooking **after tonemapping**. This is the same pattern Tonal uses for its LDR pipeline (VHS, Film Grain, etc.).

### Files to Change

**`AetherViewExtension.h`**
- Remove `PrePostProcessPass_RenderThread` override
- Add `SubscribeToPostProcessingPass` override:
```cpp
virtual void SubscribeToPostProcessingPass(
    EPostProcessingPass Pass,
    const FSceneView& View,
    FAfterPassCallbackDelegateArray& InOutPassCallbacks,
    bool bIsPassEnabled) override;
```
- The callback fires with an `FPostProcessPassInputs` that provides SceneColor as an `FScreenPassTexture`

**`AetherViewExtension.cpp`**
- Implement `SubscribeToPostProcessingPass`:
```cpp
void FAetherViewExtension::SubscribeToPostProcessingPass(
    EPostProcessingPass Pass,
    const FSceneView& View,
    FAfterPassCallbackDelegateArray& InOutPassCallbacks,
    bool bIsPassEnabled)
{
    if (Pass == EPostProcessingPass::Tonemap)
    {
        InOutPassCallbacks.Add(
            FAfterPassCallbackDelegate::CreateRaw(
                this, &FAetherViewExtension::PostTonemapPass_RenderThread));
    }
}
```
- Move the full pipeline (Tensor→Kuwahara→ZBlur→ChromaDepth→Recombine→CopyBack) into `PostTonemapPass_RenderThread`
- The callback signature is: `FScreenPassTexture PostTonemapPass_RenderThread(FRDGBuilder& GraphBuilder, const FSceneView& View, const FPostProcessMaterialInputs& Inputs)`
- Access scene color via `Inputs.GetInput(EPostProcessMaterialInput::SceneColor)`
- **Return** the modified `FScreenPassTexture` (the engine handles writeback, no `AddCopyTexturePass` needed)

### Reference Implementation

Look at Tonal's LDR pipeline in `TonalViewExtension.cpp`:
- `SubscribeToPostProcessingPass` with `EPostProcessingPass::Tonemap`
- Callback receives `FPostProcessMaterialInputs`
- Scene color is LDR [0,1] sRGB at this stage

### UV/Format Notes

After tonemapping:
- Scene color format is typically `PF_A2B10G10R10` or `PF_R8G8B8A8` (LDR formats), NOT `PF_FloatRGBA`
- Create intermediate textures with `PF_FloatRGBA` for computation precision, final output must match scene color format
- UV calculations remain the same (ViewRect-based)

---

## Phase 2: Add Edge/Outline Detection Pass (Pass 6)

### Problem

The reference oil-painting look has two components:
1. **Kuwahara** — flat color patches (we have this)
2. **Edge outlines** — dark lines at strong luminance/color gradients (we DON'T have this)

Without the outline pass, Kuwahara alone looks like stylized smoothing. The dark outlines are what sells the "painting" or "illustration" look.

### Solution

Add a new compute shader pass after Kuwahara (before Recombine) that detects strong edges via Sobel/Laplacian and outputs a darkening factor.

### New Files

**`Plugins/Aether/Shaders/AetherOutline.usf`**
```hlsl
// Sobel edge detection on luminance
// Input: FilteredTexture (post-Kuwahara)
// Output: same texture with edges darkened

// 1. Sample 3x3 neighborhood luminance
// 2. Compute Sobel magnitude: sqrt(Gx^2 + Gy^2)
// 3. Apply threshold + smoothstep for clean lines
// 4. Darken: OutputColor = InputColor * (1.0 - EdgeStrength * Strength)

// Key parameters:
//   Thickness — controls Sobel sample offset (1 = sharp, 2-3 = thicker lines)
//   Strength  — how dark the lines are (0 = invisible, 1 = full black)
//   Threshold — minimum gradient magnitude to draw a line (prevents noise)
//   EdgeColor — color of the outline (default black, but could be dark brown for watercolor)
```

The edge detection should run on the **Kuwahara output** (not the original scene color), so the lines follow the stylized patch boundaries, not the original texture detail.

### New CVars

```
r.Aether.Outline.Enabled      (int32, default=0)
r.Aether.Outline.Strength     (float, default=0.8, range 0-1)
r.Aether.Outline.Threshold    (float, default=0.05, range 0-0.5)
r.Aether.Outline.Thickness    (float, default=1.5, range 1-4)
r.Aether.Outline.ColorR       (float, default=0.05)
r.Aether.Outline.ColorG       (float, default=0.03)
r.Aether.Outline.ColorB       (float, default=0.02)
```

### Pipeline Integration

In `AetherViewExtension.cpp`, add between Kuwahara and ZBlur:

```
Pass 1: Tensor
Pass 2: Kuwahara
Pass 3: Outline (NEW — reads Kuwahara output, writes darkened edges)
Pass 4: ZBlur (optional)
Pass 5: ChromaDepth (optional)
Pass 6: Recombine
```

### Shader Class (C++)

```cpp
class FAetherOutlineCS : public FGlobalShader
{
    // Permutation: FAetherDim_HalfRes only
    // Parameters:
    //   InputTexture (SRV) — post-Kuwahara filtered color
    //   OutputTexture (UAV)
    //   Strength, Threshold, Thickness, EdgeColor
    //   OutputExtent, OutputExtentInverse
};

IMPLEMENT_GLOBAL_SHADER(FAetherOutlineCS, "/Plugin/Aether/AetherOutline.usf", "OutlineCS", SF_Compute);
```

### Settings/Panel Updates

- Add Outline fields to `FAetherSettings` struct
- Add Outline section to `AetherPanel.cpp`
- Add Outline CVars to `AetherCVars.h/.cpp`
- Update BFL (`AetherFunctionLibrary`) with outline parameter accessors
- Update presets to include outline settings

---

## Phase 3: Parameter Tuning

### Current Defaults (too conservative)

| Parameter | Current | Recommended |
|-----------|---------|-------------|
| Radius | 6 | 10 |
| Sharpness | 8.0 | 16.0 |
| EdgeFlow | 0.5 | 0.7 |
| MaxRadius | 16 | 16 |
| SectorCount | 8 | 8 |

### Preset Recommendations

Update the 6 preset `.uasset` files with better defaults:

**AP_00_Watercolor**: Radius=12, Sharpness=12, EdgeFlow=0.8, Outline.Enabled=1, Outline.Strength=0.5, Outline.Thickness=2.0, ChromaDepth.Enabled=1

**AP_01_OilPaint**: Radius=14, Sharpness=24, EdgeFlow=0.6, Outline.Enabled=1, Outline.Strength=0.9, Outline.Thickness=1.5

**AP_02_InkWash**: Radius=10, Sharpness=20, EdgeFlow=0.9, Outline.Enabled=1, Outline.Strength=1.0, Outline.Thickness=1.0, Outline.Color=(0.0, 0.0, 0.0)

**AP_03_Impressionist**: Radius=16, Sharpness=8, EdgeFlow=0.4, Outline.Enabled=0 (no outlines — pure Kuwahara patches)

**AP_04_CelShade**: Radius=8, Sharpness=32, EdgeFlow=0.3, Outline.Enabled=1, Outline.Strength=1.0, Outline.Thickness=2.5

**AP_05_Subtle**: Radius=6, Sharpness=10, EdgeFlow=0.5, Outline.Enabled=0

---

## Already Completed (This Session)

### Fix 1: Sector Accumulator Zero Initialization
- **File**: `Plugins/Aether/Shaders/AetherKuwahara.usf` (lines 111-116)
- **Change**: Replaced `float3 S0 = CenterColor` / `float W0 = 1.0` with `float3 S0 = float3(0,0,0)` / `float W0 = 0.0` for all 8 sectors
- **Why**: CenterColor initialization contaminated all sectors with the same prior, collapsing variance differences and making the soft-minimum always return approximately the original color (pass-through)
- **Status**: Applied. Shader recompilation required (`recompileshaders /Plugin/Aether/AetherKuwahara.usf` or restart editor)

### Fix 2: Preset Asset Manager Registration
- **File**: `Config/DefaultGame.ini`
- **Change**: Added `[/Script/Engine.AssetManager]` section with `AetherPreset` primary asset type registration pointing to `/Aether/Presets`
- **Why**: `UAetherPreset : UPrimaryDataAsset` requires registration; without it, UE warns on startup
- **Status**: Applied

### Architecture Verification (All Confirmed OK)
- Shader path mapping: `AddShaderSourceDirectoryMapping("/Plugin/Aether", ShaderDir)` in `Aether.cpp` StartupModule
- RDG resources: all textures via `GraphBuilder.CreateTexture/CreateSRV/CreateUAV`
- Loading phase: Runtime=`PostConfigInit`, Editor=`PostEngineInit` in `Aether.uplugin`
- GaussianWeight: already unnormalized (no PDF factor)
- Tensor eigendecomposition: correct closed-form 2x2 with Scharr gradients
- Sector angle mapping: `NormAngle = SampleAngle + PI` correctly maps `[-pi,pi]` to `[0,2pi]` with clamp for edge case

---

## Implementation Order

1. **Phase 1** (Post-Tonemapping) — do this first. It changes the visual quality of the existing Kuwahara dramatically. The variance computation on LDR values will produce the crisp flat patches seen in the reference.

2. **Phase 2** (Outline Pass) — adds the dark line art. This is what transforms "stylized smoothing" into "painting." Straightforward compute shader, no architectural complexity.

3. **Phase 3** (Presets) — update after both phases are done so presets reflect the new capabilities (outline parameters, better defaults for post-tonemapping behavior).

---

## File Map (Current)

```
Plugins/Aether/
  Aether.uplugin                              — Plugin descriptor
  Shaders/
    AetherCommon.ush                          — Shared utilities (depth, luma, Scharr, Gaussian)
    AetherTensor.usf                          — Pass 1: Structure tensor
    AetherKuwahara.usf                        — Pass 2: Anisotropic Kuwahara (zero-init fix applied)
    AetherZBlur.usf                           — Pass 3: Separable depth blur
    AetherChromaDepth.usf                     — Pass 4: Atmospheric chroma shift
    AetherRecombine.usf                       — Pass 5: Final composite + debug vis
    AetherOutline.usf                         — Pass 6: Edge outline (TO CREATE)
  Content/Presets/
    AP_00_Watercolor.uasset                   — Watercolor preset
    AP_01_OilPaint.uasset                     — Oil paint preset
    AP_02_InkWash.uasset                      — Ink wash preset
    AP_03_Impressionist.uasset                — Impressionist preset
    AP_04_CelShade.uasset                     — Cel shade preset
    AP_05_Subtle.uasset                       — Subtle preset
  Source/Aether/
    Aether.h / .cpp                           — Module startup, shader dir mapping, view extension create
    AetherViewExtension.h / .cpp              — Core rendering: 5-pass pipeline (REFACTOR for post-tonemap)
    AetherCVars.h / .cpp                      — All CVars (ADD outline CVars)
    AetherSettings.h / .cpp                   — FAetherSettings USTRUCT (ADD outline fields)
    AetherPreset.h                            — UAetherPreset : UPrimaryDataAsset
    AetherComponent.h / .cpp                  — Component for actor attachment
    AetherVolume.h / .cpp                     — Volume for spatial blending
    AetherFunctionLibrary.h / .cpp            — Blueprint function library
    AetherTypes.h                             — Enums and type definitions
  Source/AetherEditor/
    AetherEditor.h / .cpp                     — Editor module, track editor registration
    AetherPanel.h / .cpp                      — Slate editor UI panel (ADD outline section)
    Sequencer/                                — Sequencer integration files
```
