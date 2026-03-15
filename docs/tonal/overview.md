# Tonal — Overview

## Architecture

Tonal is a multi-stage compute pipeline running before UE's built-in tone mapper.

```
UTonalSubsystem  (UEngineSubsystem)
    └── FTonalViewExtension  (FSceneViewExtensionBase)
            └── PrePostProcessPass_RenderThread()
                    ├── Stage 1: FilmStock.usf      — emulsion curve + color response
                    ├── Stage 2: Grain.usf           — structured film grain
                    ├── Stage 3: Halation.usf        — red-channel bloom from film base
                    ├── Stage 4: VHS.usf             — analog tape artifacts (optional)
                    └── Stage 5: LUT.usf             — creative LUT application
```

Each stage reads from and writes to ping-pong HDR buffers. Disabled stages are skipped entirely — no dummy dispatch.

## Pipeline Stages

### Film Stock Emulation

The core stage applies a per-channel response curve derived from densitometric measurements of real emulsion stocks. Highlights roll off with characteristic shoulder compression, and shadows lift slightly to mimic base fog in photographic film.

Four built-in presets ship with Tonal:

| Preset | Character |
|--------|-----------|
| `Kodak Vision3 500T` | Warm shadows, cool highlights, high saturation |
| `Fuji Eterna 500` | Neutral, slightly desaturated, cinematic |
| `Kodak 2383 Print` | High contrast print stock look |
| `Monochrome` | Luminance-only silver halide response |

Custom curves can be authored in the Tonal settings asset.

### Film Grain

Structured grain based on a blue-noise texture tiled with per-frame temporal offset. Grain size, intensity, and luma-dependency are all tunable. High-ISO grain clusters at shadow values; low-ISO grain is nearly imperceptible in highlights.

### Halation

Simulates the red-channel light bloom that occurs when intense highlights scatter through the film base. Implemented as a separable Gaussian applied only to the red channel above a configurable threshold. The spread and intensity are independently controllable.

### VHS Mode

An optional analog artifact stage that adds:
- Luma/chroma separation with independent noise fields
- Horizontal scanline shimmer
- Occasional signal dropout (brief horizontal white bars)
- Color bleeding at high-contrast edges

### Creative LUT

A final 33-cube LUT pass for creative grade. Any standard `.cube` LUT can be imported and assigned via the settings asset.

## Key Concepts

| Concept | Detail |
|---------|--------|
| **HDR-first** | All stages run in scene linear space before tone mapping |
| **Stage permutations** | Each stage is a shader permutation, disabled stages cost zero |
| **CVar authority** | CVars are the single source of truth at render time |
| **Sequencer support** | All parameters are Sequencer-animatable via `UTonalSequencerTrack` |
