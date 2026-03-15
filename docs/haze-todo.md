# Haze — Stylized Atmospheric Fog Plugin

## Vision
Compute-shader-driven stylized fog system for horror, dreamscapes, and atmospheric worlds.
NOT a realistic fog enhancer (that's Vapor). Haze is animated, artistic, over-the-top.
Every effect is permutation-gated for dead code elimination when disabled.

## Design Scenarios (driving feature requirements)
| Scenario | Key Features Used |
|---|---|
| Silent Hill fog wall | Radial falloff, animated noise, distance field crawl, gray color |
| Boss arena | Radial clear zone, pulsing, colored (red/green), void absorption |
| Dream sequence | Color shift over time, soft tendrils, ethereal wisp layer |
| Underground / cave | Ground fog (height-locked), floor pooling, dripping density |
| Swamp / forest | Low-lying ground fog, wisps rising, green/brown tint |
| Corruption spread | Directional creep from world point, dark tendrils, absorption |
| Underwater | Blue-green tint, heavy near-cam density, slow noise scroll |
| Void / abyss | Absorption mode (darkens instead of glows), hard distance cutoff |
| Paranormal reveal | Distance field "reveal" mode — surfaces emerge as silhouettes |
| Level transition | Full-screen sweep, animated wipe via directional density |

## Architecture Overview
Same pipeline pattern as Vapor (4-pass compute):
1. **HazeSetup** — Compute fog density from noise + radial + height + DF (NOT from CalculateHeightFog)
2. **HazeDownsample** — Kawase blur chain (quality-tiered tap counts)
3. **HazeUpsample** — Bilateral upsample with depth-aware filtering
4. **HazeRecombine** — Composite to scene (additive OR absorption blend, + temporal)

Shared include: `HazeCommon.ush` (Reinhard, bounds check, noise functions)

## Quality Tiers
| Feature | Low (0) | Medium (1) | High (2) | Cinematic (3) |
|---|---|---|---|---|
| Noise octaves | 1 | 2 | 2 | 3 |
| Distance field | No | No | Yes (perm) | Yes |
| Secondary noise (wisps) | No | Optional | Yes | Yes |
| Downsample taps | 4-tap | 8-tap | 13-tap | 13-tap |
| Temporal | No | Yes | Yes | Yes |
| Self-shadow | No | No | Yes | Yes |
| Edge vignette | No | No | Optional | Yes |
| Chromatic | No | No | No | Optional |
| Half-res setup | Always | Always | Optional | Optional |

---

## Phase 1: Plugin Scaffold
Create the empty plugin structure, module registration, build files.

### Files to create:
- `Plugins/Haze/Haze.uplugin` — descriptor (Runtime + Editor modules)
- `Plugins/Haze/Source/Haze/Haze.h` + `Haze.cpp` — runtime module (FDefaultGameModuleImpl; view extension creation deferred to PostEngineInit)
- `Plugins/Haze/Source/Haze/Haze.Build.cs` — deps: Core, CoreUObject, Engine, RenderCore, Renderer, RHI, Projects; PrivateIncludePaths for Renderer/Private + Renderer/Internal
- `Plugins/Haze/Source/HazeEditor/HazeEditor.h` + `HazeEditor.cpp` — editor module stub
- `Plugins/Haze/Source/HazeEditor/HazeEditor.Build.cs` — deps: Slate, SlateCore, UnrealEd, InputCore, WorkspaceMenuStructure, ToolMenus, AssetTools, AssetRegistry + Haze runtime module
- `Plugins/Haze/Shaders/HazeCommon.ush` — Reinhard3/InvReinhard3, Luminance, KarisWeight, IsOutOfBounds, noise functions

### Key decisions:
- Runtime module LoadingPhase: `PostConfigInit`
- Editor module LoadingPhase: `PostEngineInit`
- Module API macro: `HAZE_API`

---

## Phase 2: CVars + Settings

### CVars (`HazeCVars.h/.cpp`) — all `r.Haze.*`, all `HAZE_API`:

**Core (6)**
- `r.Haze.Enabled` (bool, default 1)
- `r.Haze.Intensity` (float, 0-5, default 1.0)
- `r.Haze.Quality` (int, 0-3, default 2) — Low/Med/High/Cinematic
- `r.Haze.PassAmount` (int, 2-6, default 4) — blur chain depth
- `r.Haze.HalfRes` (bool, default 1) — setup pass at half res
- `r.Haze.Format` (int, 0-1, default 0) — 0=R11G11B10F, 1=FloatRGBA

**Radial Density (5)**
- `r.Haze.InnerRadius` (float, 0-10000, default 200) — clear zone around camera (cm)
- `r.Haze.OuterRadius` (float, 0-50000, default 5000) — fog wall distance (cm)
- `r.Haze.FalloffCurve` (float, 0.1-10, default 2.0) — power curve for radial ramp
- `r.Haze.BaseDensity` (float, 0-1, default 0.8) — fog density at/beyond outer radius
- `r.Haze.NearFade` (float, 0-500, default 50) — fade fog out within this cm of camera

**Height Mask (6)**
- `r.Haze.HeightEnabled` (bool, default 0)
- `r.Haze.HeightMode` (int, 0-1, default 0) — 0=ground fog (below), 1=ceiling fog (above)
- `r.Haze.HeightBase` (float, -10000 to 10000, default 0) — world Z of fog floor/ceiling
- `r.Haze.HeightFalloff` (float, 0-5000, default 500) — vertical fade distance (cm)
- `r.Haze.HeightDensity` (float, 0-1, default 1.0) — density at the base height
- `r.Haze.HeightNoise` (float, 0-1, default 0.2) — noise modulation on height boundary

**Primary Noise (7)**
- `r.Haze.NoiseEnabled` (bool, default 1)
- `r.Haze.NoiseScale` (float, 0.0001-0.1, default 0.002) — world-space frequency (1/cm)
- `r.Haze.NoiseSpeed` (float, 0-100, default 10) — scroll speed (cm/s)
- `r.Haze.NoiseAmplitude` (float, 0-1, default 0.4) — density modulation depth
- `r.Haze.NoiseOctaves` (int, 1-3, default 2) — auto-capped by quality tier
- `r.Haze.NoiseWindX/Y/Z` (float, -1 to 1, default 1/0/0) — scroll direction bias

**Secondary Noise / Wisps (5)**
- `r.Haze.WispsEnabled` (bool, default 0)
- `r.Haze.WispsScale` (float, 0.0001-0.05, default 0.0005) — larger = finer wisps
- `r.Haze.WispsStretch` (float, 1-20, default 8) — anisotropic stretch ratio
- `r.Haze.WispsSpeed` (float, 0-50, default 3) — independent scroll speed
- `r.Haze.WispsIntensity` (float, 0-1, default 0.3) — blend with primary noise

**Pulsing (4)**
- `r.Haze.PulseEnabled` (bool, default 0)
- `r.Haze.PulseFrequency` (float, 0.01-5, default 0.15) — Hz
- `r.Haze.PulseAmplitude` (float, 0-0.5, default 0.1) — density oscillation range
- `r.Haze.PulseAsymmetry` (float, 0-1, default 0) — 0=sine, 1=breathing (slow in, fast out)

**Directional Creep (5)**
- `r.Haze.CreepEnabled` (bool, default 0)
- `r.Haze.CreepDirectionX/Y/Z` (float, -1 to 1, default 1/0/0)
- `r.Haze.CreepSpeed` (float, 0-200, default 30) — cm/s
- `r.Haze.CreepIntensity` (float, 0-1, default 0.5) — blend

**Color (12)**
- `r.Haze.ColorR/G/B` (float, 0-1, default 0.5/0.5/0.5) — base fog color
- `r.Haze.FarColorR/G/B` (float, 0-1, default 0.3/0.3/0.35) — color at fog wall
- `r.Haze.ColorGradient` (float, 0-1, default 0) — lerp between near/far color by distance
- `r.Haze.ColorShiftEnabled` (bool, default 0)
- `r.Haze.ColorShiftSpeed` (float, 0-1, default 0.05) — hue rotation speed
- `r.Haze.ColorShiftRange` (float, 0-1, default 0.1) — hue offset range
- `r.Haze.AbsorptionEnabled` (bool, default 0) — void mode: fog darkens instead of lightens
- `r.Haze.AbsorptionStrength` (float, 0-1, default 0.5) — how much scene is darkened

**Distance Field (4)**
- `r.Haze.DFEnabled` (bool, default 0)
- `r.Haze.DFMode` (int, 0-1, default 0) — 0=crawl (denser near surfaces), 1=reveal (thinner near surfaces)
- `r.Haze.DFInfluence` (float, 0-500, default 100) — influence radius in cm
- `r.Haze.DFStrength` (float, 0-1, default 0.5) — effect strength

**Self-Shadow (4)**
- `r.Haze.ShadowEnabled` (bool, default 0)
- `r.Haze.ShadowIntensity` (float, 0-1, default 0.3) — shadow darkening
- `r.Haze.ShadowSoftness` (float, 0-1, default 0.5) — shadow edge softness
- `r.Haze.ShadowColorR/G/B` (float, 0-1, default 0/0/0.05) — shadow tint (blueish default)

**Edge Vignette (4)**
- `r.Haze.VignetteEnabled` (bool, default 0)
- `r.Haze.VignetteIntensity` (float, 0-1, default 0.3) — fog density boost at edges
- `r.Haze.VignetteInnerRadius` (float, 0-1, default 0.4) — screen-space inner clear
- `r.Haze.VignetteOuterRadius` (float, 0-1, default 0.9) — screen-space full fog

**Light Scatter (4)**
- `r.Haze.ScatterEnabled` (bool, default 0)
- `r.Haze.ScatterIntensity` (float, 0-3, default 1.0)
- `r.Haze.ScatterG` (float, -0.9 to 0.9, default 0.35) — HG asymmetry
- `r.Haze.ScatterAmbient` (float, 0-1, default 0.1) — ambient scatter floor

**Temporal (2)**
- `r.Haze.Temporal` (bool, default 1)
- `r.Haze.TemporalBlend` (float, 0-0.98, default 0.9)

**Depth Aware (2)**
- `r.Haze.DepthAware` (bool, default 1)
- `r.Haze.DepthSensitivity` (float, 0-100, default 10)

**Total: ~70 CVars**

### Settings (`HazeSettings.h/.cpp`):
- `USTRUCT(BlueprintType) FHazeSettings` — mirrors all CVars as UProperties
- `void ApplyToCVars() const`
- `void ReadFromCVars()`
- `static FHazeSettings Lerp(const FHazeSettings& A, const FHazeSettings& B, float Alpha)`
- Bools snap at Alpha >= 0.5; floats/ints interpolate linearly

---

## Phase 3: View Extension + Render Pipeline

### `HazeViewExtension.h/.cpp`:
- `FHazeViewExtension : FSceneViewExtensionBase`
- Hook: `PrePostProcessPass_RenderThread`
- `IsActiveThisFrame()` — check `r.Haze.Enabled`

### Render target management:
- `GetIntermediateFormat()` based on `r.Haze.Format` CVar
- Mip chain allocation for downsample/upsample
- Temporal history buffer (persistent, not RDG-transient)

### Pass dispatch order:
1. `RenderSetup()` — compute density at half or full res
2. `RenderDownsampleChain()` — N progressive halving passes
3. `RenderUpsampleChain()` — N-1 doubling passes
4. `RenderRecombine()` — composite to SceneColor + temporal blend

### Shader declarations (FGlobalShader subclasses):
- `FHazeSetupCS` — permutations: HAZE_NOISE, HAZE_WISPS, HAZE_HEIGHT, HAZE_DF, HAZE_SHADOW, HAZE_SCATTER, HAZE_PULSE, HAZE_CREEP, HAZE_VIGNETTE, HAZE_ABSORPTION
- `FHazeDownsampleCS` — permutation: HAZE_QUALITY (0-3)
- `FHazeUpsampleCS` — permutations: HAZE_QUALITY, HAZE_DEPTH_AWARE
- `FHazeRecombineCS` — permutations: HAZE_TEMPORAL, HAZE_ABSORPTION

### Uniform buffer (`FHazeParameters`):
- All per-frame scalars packed into a single UB
- Sun direction from directional light (for shadow + scatter)
- Time (real-time seconds for noise animation)
- GDF parameters (when DF enabled)

---

## Phase 4: Setup Shader — Density

### `HazeSetup.usf`:

**World position reconstruction** (same as Vapor):
```
DeviceZ → SceneDepth → ScreenPos → TranslatedWorldPos
WorldPositionRelativeToCamera = TranslatedWorldPos - CameraOrigin
```

**Radial density**:
```
Distance = length(WorldPositionRelativeToCamera)
RadialDensity = smoothstep(InnerRadius, OuterRadius, Distance) * BaseDensity
Apply NearFade: *= smoothstep(0, NearFade, Distance)
Apply FalloffCurve: pow(t, FalloffCurve)
```

**Height mask** (permutation `HAZE_HEIGHT`):
```
WorldZ = TranslatedWorldPos.z - PreViewTranslation.z  // absolute world Z
HeightFactor = ground mode: smoothstep(HeightBase + HeightFalloff, HeightBase, WorldZ)
               ceiling mode: smoothstep(HeightBase - HeightFalloff, HeightBase, WorldZ)
Density *= HeightFactor * HeightDensity
```

**Sky pixel guard**: `DeviceZ < 1e-7` — no fog on sky pixels (same as Vapor)

**Output**: `float4(FogColor * Density, Density)`

---

## Phase 5: Noise System

### Procedural 3D noise (in `HazeCommon.ush`):
- Implement gradient noise (Simplex or Perlin) directly in HLSL
- No texture dependency — pure math, works at any scale
- GPU-efficient: use bit manipulation for hash, smooth interpolation

### Primary noise application:
```
NoisePos = WorldPos * NoiseScale + Time * NoiseSpeed * WindDir
N = fbm(NoisePos, Octaves)  // octave count from quality tier cap
Density *= lerp(1.0, N, NoiseAmplitude)
```

### Secondary noise / Wisps (permutation `HAZE_WISPS`):
```
WispPos = WorldPos * WispsScale
WispPos.xz *= WispsStretch  // stretch along XZ plane
WispN = noise(WispPos + Time * WispsSpeed * float3(0,1,0))
Density += WispN * WispsIntensity * RadialDensity
```

### Pulsing (permutation `HAZE_PULSE`):
```
PulseT = Time * PulseFrequency * 2PI
if Asymmetry > 0:
    wave = sin(PulseT)
    wave = sign(wave) * pow(abs(wave), lerp(1, 0.3, Asymmetry))  // breathing
else:
    wave = sin(PulseT)
Density *= 1.0 + wave * PulseAmplitude
```

### Directional Creep (permutation `HAZE_CREEP`):
```
CreepDot = dot(normalize(WorldPositionRelativeToCamera), normalize(CreepDirection))
CreepWave = noise(WorldPos * 0.001 + Time * CreepSpeed * CreepDirection)
CreepFactor = saturate(CreepDot * 0.5 + 0.5) * CreepWave
Density += CreepFactor * CreepIntensity * BaseDensity
```

---

## Phase 6: Blur Chain

### `HazeDownsample.usf`:
- Identical structure to Vapor's VaporDownsample.usf
- Quality 0: 4-tap box, Quality 1-2: 8-tap, Quality 3: 13-tap COD:AW
- Karis luminance weighting for anti-firefly

### `HazeUpsampleCombine.usf`:
- Quality 0-1: 5-tap cross, Quality 2-3: 9-tap 3x3 tent
- `HAZE_DEPTH_AWARE` permutation for bilateral depth rejection
- Radius parameter for blur spread

### Both adapted from Vapor with `Haze` prefix naming.

---

## Phase 7: Recombine + Temporal

### `HazeRecombine.usf`:
- Two composite modes:
  - **Additive** (default): `SceneColor += BlurredInscatter` — fog glows/brightens
  - **Absorption** (permutation `HAZE_ABSORPTION`): `SceneColor *= (1 - AbsorptionStrength * Density)` then `+= tinted inscatter` — fog darkens AND tints

### Temporal (permutation `HAZE_TEMPORAL`):
- Same pattern as Vapor: history buffer, ClipToPrevClip reprojection, neighborhood clamp
- Critical for noise stability — without temporal, animated noise shimmers/crawls too harshly

### Color processing:
- Distance gradient: `FinalColor = lerp(NearColor, FarColor, ColorGradient * distanceFraction)`
- Color shift (runtime, not permutation — cheap): hue rotation via RGB→HSV→RGB
- Edge vignette (permutation `HAZE_VIGNETTE`): `Density += VignetteBoost * screenEdgeMask`

---

## Phase 8: Distance Field Integration

### Permutation: `HAZE_DF` (bool)

### C++ setup:
- Include `GlobalDistanceFieldParameters.h`
- Use `FGlobalDistanceFieldParameters2` via `SetupGlobalDistanceFieldParameters_Minimal()`
- Bind as nested shader parameter struct in `FHazeSetupCS::FParameters`
- Get `FGlobalDistanceFieldParameterData` from `FViewInfo` in PrePostProcessPass

### HLSL:
```
#if HAZE_DF
#include "/Engine/Private/DistanceField/GlobalDistanceFieldShared.ush"

float SurfaceDist = GetDistanceToNearestSurfaceGlobal(TranslatedWorldPos);
float DFMask = smoothstep(0, DFInfluence, SurfaceDist);

if (DFMode == 0)  // Crawl: denser near surfaces
    Density *= lerp(1.0 + DFStrength, 1.0, DFMask);
else              // Reveal: thinner near surfaces
    Density *= lerp(1.0 - DFStrength, 1.0, 1.0 - DFMask);
#endif
```

### Performance note:
- GDF sampling is the most expensive per-pixel operation
- At half-res, cost is 1/4 of full-res — always recommend half-res when DF enabled
- Quality Low/Medium: permutation OFF (zero cost)

---

## Phase 9: Self-Shadow + Light Scatter

### Self-Shadow (permutation `HAZE_SHADOW`):
```
// Cheap approximation: offset world pos along light dir, resample density
float3 ShadowSamplePos = WorldPos + SunDirection * ShadowSoftness * 200.0;
float ShadowNoise = noise(ShadowSamplePos * NoiseScale);
float ShadowFactor = lerp(1.0, ShadowNoise, ShadowIntensity);
FogColor = lerp(FogColor, ShadowColor, (1.0 - ShadowFactor) * ShadowIntensity);
```

### Light Scatter (permutation `HAZE_SCATTER`):
```
// Reuse HG phase function from HazeCommon.ush
float CosTheta = dot(ViewDir, SunDirection);
float Phase = HenyeyGreenstein(CosTheta, ScatterG) * 4PI;
float PhaseScale = lerp(1.0, Phase, ScatterIntensity) + ScatterAmbient;
FogColor *= PhaseScale;
```

---

## Phase 10: Volume + Subsystem

### `HazeVolume.h/.cpp` — `AHazeVolume : AVolume`:
- `bEnabled`, `bUnbound`, `Priority`, `BlendRadius`, `BlendWeight`
- `FHazeSettings Settings`
- `TSoftObjectPtr<UHazePreset> Preset`
- `GetEffectiveSettings()`, `GetBlendFactor(CameraLocation)`
- `SettingsVersion` for edit detection

### `HazeVolumeSubsystem.h/.cpp` — `UHazeVolumeSubsystem : UTickableWorldSubsystem`:
- Baseline snapshot/restore
- Per-frame blend evaluation (same algorithm as Vapor/PostProcessVolume)
- Priority sorting, distance-weighted blend

---

## Phase 11: Editor Panel

### `HazePanel.h/.cpp` — `SHazePanel : SCompoundWidget`:
- Uses TABS (not spaces) — must use Bash/Python for file edits
- Reuse MakeFloatRow/MakeBoolRow/MakeIntRow/MakeRGBRow/MakeEnumRow/MakeSection helpers

### Sections:
1. **Enable + Quality** — master toggle, quality preset dropdown
2. **Fog Shape** — radial (inner/outer radius, falloff curve, near fade), height mask
3. **Animation** — primary noise, wisps, pulsing, directional creep
4. **Color** — base color, far color, gradient, color shift, absorption
5. **Surface Interaction** — distance field enable/mode/influence/strength
6. **Lighting** — self-shadow, light scatter
7. **Screen Effects** — edge vignette
8. **Performance** — half-res, format, temporal, depth-aware, pass amount
9. **Presets** — combo box, save/load/delete

---

## Phase 12: Presets

### `UHazePreset : UDataAsset`:
- Stores `FHazeSettings`
- Named presets shipped with plugin:

| Preset | Description |
|---|---|
| Silent Hill | Thick gray wall, animated noise, outer radius 4000, high density |
| Boss Arena | Red tint, pulsing, tight inner clear zone, absorption 0.3 |
| Dream | Ethereal white/blue, color shift on, wisps on, soft density |
| Cave | Ground fog mode, height 100, green-gray, low noise speed |
| Swamp | Low ground fog, wisps enabled, stretched, murky green-brown |
| Corruption | Directional creep, dark purple, absorption on, tendrils |
| Abyss | Full absorption, near-black, hard outer radius, no noise |
| Underwater | Blue-green, high density, slow noise, heavy near-cam |
| Paranormal | DF reveal mode, low density, subtle wisps, cold blue |
| Transition | Full screen, high density, fast creep, white — for level wipes |

---

## Phase 13: Blueprint Function Library

### `UHazeFunctionLibrary : UBlueprintFunctionLibrary`:
- `SetEnabled(bool)`, `IsEnabled() -> bool`
- `SetQuality(int)`, `ApplyPreset(Name)`
- `SetFloat(CVar, Value)`, `SetBool(CVar, Value)`
- `SetFogColor(R, G, B)`, `SetFarColor(R, G, B)`
- `SetRadialFalloff(Inner, Outer, Curve)`
- `SetNoiseParams(Scale, Speed, Amplitude, Octaves)`
- `SetPulse(Enabled, Freq, Amplitude, Asymmetry)`
- `SetCreep(Enabled, DirX, DirY, DirZ, Speed, Intensity)`
- `GetParameterInfo() -> TArray<FHazeParameterInfo>`
- `DumpCurrentSettings()`

---

## Phase 14: Sequencer Integration

### Snapshot-key model (same as Lucent):
- `FHazeSettingsKey` = `FFrameNumber` + `FHazeSettings`
- `UHazeSection` — snapshot keys, `Evaluate()` lerps
- `UMovieSceneHazeTrack` — master track (no binding)
- `UHazeTrackInstance` — `OnAnimate()` calls `Evaluate(T).ApplyToCVars()`
- `FHazeTrackEditor` — handles Add Track, Add Key, Preset drag-drop

### Volume track (object-bound):
- `UHazeVolumeSection/Track/TrackInstance` — same pattern as Vapor/Lucent/Tonal volume tracks
- `OnAnimate()` writes `Volume->Settings = Evaluate(T); Volume->SettingsVersion++`

---

## Implementation Order

| Step | Phase | Depends On | Est. Files |
|---|---|---|---|
| 1 | Phase 1: Scaffold | — | 7 |
| 2 | Phase 2: CVars + Settings | Phase 1 | 4 |
| 3 | Phase 3: View Extension skeleton | Phase 2 | 2 |
| 4 | Phase 4: Setup shader (radial + height) | Phase 3 | 1 |
| 5 | Phase 6: Blur chain (down + up) | Phase 4 | 2 |
| 6 | Phase 7: Recombine + temporal | Phase 5 | 1 |
| 7 | **CHECKPOINT: basic fog visible on screen** | — | — |
| 8 | Phase 5: Noise system | Phase 4 | 1 (edit HazeSetup + HazeCommon) |
| 9 | Phase 8: Distance field | Phase 4 | 1 (edit HazeSetup + view ext) |
| 10 | Phase 9: Shadow + scatter | Phase 4 | 1 (edit HazeSetup) |
| 11 | Phase 7 addendum: absorption + color + vignette | Phase 6 | edit Recombine + Setup |
| 12 | **CHECKPOINT: full shader feature set** | — | — |
| 13 | Phase 10: Volume + subsystem | Phase 2 | 4 |
| 14 | Phase 11: Editor panel | Phase 2 | 2 |
| 15 | Phase 12: Presets | Phase 2 | 1 + preset assets |
| 16 | Phase 13: BFL | Phase 2 | 2 |
| 17 | Phase 14: Sequencer | Phase 2, 10 | 10 |
| 18 | **CHECKPOINT: shipping feature complete** | — | — |

## Total estimated files: ~38-42

## Performance Budget
- **Target**: < 0.5ms at 1080p on RTX 3060 (High quality)
- Setup at half-res = 1/4 pixel count
- Noise is ALU-bound (no texture fetches for primary noise)
- GDF sampling is the most expensive op — permutation-gated, half-res only
- Blur chain is identical cost to Vapor (well-proven)
- Temporal adds one history read + one write per frame (cheap)
