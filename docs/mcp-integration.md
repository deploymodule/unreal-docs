# PluginDepot MCP Integration

## What We're Doing (And What We're Not)

We are **NOT** building a new MCP server. We already have the Unreal Engine Pro Tools MCP
(`unreal-handshake`). The goal is to make Vapor, Lucent, and Tonal **discoverable and
controllable** by any LLM that connects to that existing MCP.

## Why This Works Without Building Anything New

The existing MCP can already:
- Execute console commands via `generate_blueprint_logic` (which can emit `Execute Console Command` nodes)
- Read/write Data Asset properties via `edit_data_asset_defaults`
- Search ALL Blueprint content in the project via `scan_and_index_project_blueprints` → `get_relevant_blueprint_context`

The gap: the MCP has no idea what CVars `r.Vapor.*`, `r.Lucent.*`, or `r.Tonal.*` exist,
what they mean, or what ranges are valid. Fix that, and the MCP can control them fully.

## The Two Pieces Needed Per Plugin

### 1. Blueprint Function Library (C++ — ~100 lines each)
A `UCLASS(BlueprintType)` with `BlueprintCallable` static methods:
- `SetEnabled(bool)` / `IsEnabled()`
- `ApplyPreset(int32 PresetIndex)` / `GetPresetName(int32)`
- `SetParameter(FName CVar, float Value)` — generic CVar setter
- `GetParameterInfo()` — returns a human-readable string listing all CVars, types, ranges, defaults
- `GetCurrentSettings()` — dumps current CVar state as a JSON string

Once created, `scan_and_index_project_blueprints` will index them. Then
`get_relevant_blueprint_context("how do I control fog intensity in Vapor")` will return
the correct function signatures. `generate_blueprint_logic` can then call them.

### 2. Plugin Manifest JSON (Data file — ~200-300 lines each)
Placed at `Plugins/<Name>/plugindepot-mcp.json`. Documents every CVar:
```json
{
  "plugin": "Vapor",
  "description": "Enhanced fog light scattering via compute-shader pipeline",
  "cvar_prefix": "r.Vapor",
  "parameters": [
    {
      "cvar": "r.Vapor.Enabled",
      "type": "int32",
      "range": [0, 1],
      "default": 1,
      "description": "Master enable/disable for Vapor fog scattering"
    },
    {
      "cvar": "r.Vapor.Intensity",
      "type": "float",
      "range": [0.0, 10.0],
      "default": 1.0,
      "description": "Global intensity multiplier for fog scattering effect"
    }
    // ... all CVars
  ],
  "presets": [
    { "index": 0, "name": "Custom", "description": "User-defined settings" },
    { "index": 1, "name": "Light Haze", "description": "Subtle atmospheric haze" }
  ]
}
```

This manifest is a static reference file any LLM can read as context before issuing commands.

### Optional — 3. Master Context File (Already doing this by writing this doc)
`docs/mcp-integration.md` (this file) serves as the LLM orientation doc.

---

## Todo List

### Phase 1: Vapor Blueprint Function Library

- [ ] Create `Plugins/Vapor/Source/Vapor/VaporFunctionLibrary.h`
  - `UCLASS()` with `BlueprintType`, `Blueprintable`
  - Static `BlueprintCallable` methods: `SetEnabled`, `IsEnabled`, `ApplyPreset`,
    `GetPresetCount`, `GetPresetName`, `SetFloat`, `SetBool`, `GetParameterInfo`, `DumpCurrentSettings`
  - `GetParameterInfo()` returns a single formatted string — the "cheat sheet" for any LLM

- [ ] Create `Plugins/Vapor/Source/Vapor/VaporFunctionLibrary.cpp`
  - `GetParameterInfo()` returns a hardcoded string documenting every CVar grouping,
    name, type, range, and default
  - `DumpCurrentSettings()` iterates all CVars via `IConsoleManager::Get().ForEachConsoleObject`
    filtered to `r.Vapor.*`, returns as formatted key=value string
  - `SetFloat(FString CVarName, float Value)` / `SetBool(FString CVarName, bool Value)` —
    validates name starts with `r.Vapor.`, then calls `SetValueWithPriority`
  - `ApplyPreset(int32)` — copies preset switch logic from existing VaporSettings

- [ ] Add `VaporFunctionLibrary.cpp` to `Vapor.Build.cs` source list (if explicit)

- [ ] Create `Plugins/Vapor/plugindepot-mcp.json`
  - All ~55 Vapor CVars from `VaporCVars.h` with types, ranges, defaults, descriptions
  - Grouped sections matching the CVar header: Core, Density Gating, Phase Function,
    Chromatic Scattering, Color, Artistic, Depth-Aware, Fog Sources, Volumetric Quality,
    Spotlight Shafts, Additional Lights, Temporal, Performance, Debug
  - Preset list with names and descriptions

---

### Phase 2: Lucent Blueprint Function Library

- [ ] Create `Plugins/Lucent/Source/Lucent/LucentFunctionLibrary.h`
  - Same pattern as Vapor
  - Extra methods: `SetBloomLayerWeight(int32 Layer, float Weight)`,
    `SetBloomLayerTint(int32 Layer, FLinearColor Tint)`,
    `SetHueSelectSlot(int32 Slot, ...)` — helpers for the multi-slot CVars

- [ ] Create `Plugins/Lucent/Source/Lucent/LucentFunctionLibrary.cpp`
  - `GetParameterInfo()` covers all modules: DOF, Bloom (6 layers), Halation, Streaks,
    Dirt Mask, Glow Pipeline, Distortion, Chromatic Aberration, Misregistration,
    Vignette, Film Grain, Gate Weave

- [ ] Create `Plugins/Lucent/plugindepot-mcp.json`
  - All ~100+ Lucent CVars with types, ranges, defaults, descriptions
  - Module groupings: DOF, Bloom, Halation, Streaks, Dirt, Glow, Distortion, CA,
    Misregistration, Vignette, Grain, Weave

---

### Phase 3: Tonal Blueprint Function Library

- [ ] Create `Plugins/Tonal/Source/Tonal/TonalFunctionLibrary.h`
  - Same pattern
  - Extra: `ApplyFilmStockPreset(int32)`, `SetWhiteBalance(int32 Preset, float TempK)`,
    `SetHueSelectSlot(int32 Slot, ...)`, `EnableVHS(bool)`, `SetVHSParameter(FString, float)`

- [ ] Create `Plugins/Tonal/Source/Tonal/TonalFunctionLibrary.cpp`
  - `GetParameterInfo()` covers all modules: Shaper, MicroContrast, Sharpen, SplitTone,
    SubjectEmphasis, BlackFloor, MidtoneGlow, SkinSoftener, HighlightDesat, MidtoneSat,
    HueShift, DepthFade, BleachBypass, FilmStock (with Presets/WB/PushPull),
    HueSelect (8 slots), DepthWB, LocalTone, LightWrap, FilmVignette, LGG, VHS, XPro

- [ ] Create `Plugins/Tonal/plugindepot-mcp.json`
  - All ~150+ Tonal CVars (including 8-slot arrays and VHS animation params)
  - Film Stock preset table: Custom, Warm Tungsten 500T, Cool Daylight 250D,
    Natural Portrait 400, Vivid Reversal, High Contrast Noir, Faded Print
  - White Balance preset table: Custom, Tungsten 3200K, Fluorescent 4000K,
    Daylight 5600K, Overcast 7500K

---

### Phase 4: Integration Verification

- [ ] Run `scan_and_index_project_blueprints` in-editor after Phase 1–3
  to verify all three function libraries appear in the index

- [ ] Test: ask the MCP `get_relevant_blueprint_context("how do I enable fog scattering")`
  → should return `VaporFunctionLibrary::SetEnabled` info

- [ ] Test: use `generate_blueprint_logic` to call `VaporFunctionLibrary::SetFloat`
  with `"r.Vapor.Intensity"`, `2.0` → confirm fog changes in viewport

- [ ] Test: `get_relevant_blueprint_context("apply Tonal VHS tape look")`
  → should return VHS CVar info and `TonalFunctionLibrary::EnableVHS`

- [ ] Test: `get_relevant_blueprint_context("cinematic bloom in Lucent")`
  → should return Lucent bloom module info

---

### Phase 5: LLM Orientation Prompt Snippets (Optional Polish)

- [ ] Write `docs/mcp-prompts.md` — a set of ready-to-paste prompts for common tasks:
  - "Set up a cinematic look with Tonal Film Stock Warm Tungsten + Lucent Halation"
  - "Enable subtle fog scattering in Vapor for an indoor scene"
  - "Make the scene look like a VHS home video"
  - "Dial in Lucent DOF with hexagonal bokeh"
  These demonstrate how an LLM should chain calls across multiple plugins.

---

## How an LLM Uses These Plugins After This Work

```
User: "Make this look like old VHS footage"

LLM using the MCP:
1. get_relevant_blueprint_context("VHS tape emulation Tonal")
   → returns TonalFunctionLibrary::EnableVHS, SetVHSParameter docs
2. generate_blueprint_logic: call TonalFunctionLibrary::EnableVHS(true)
3. generate_blueprint_logic: SetVHSParameter("r.Tonal.VHS.ChromaShift", 0.8)
4. generate_blueprint_logic: SetVHSParameter("r.Tonal.VHS.Scanlines", 0.6)
5. generate_blueprint_logic: SetVHSParameter("r.Tonal.VHS.LumaCrush", 0.4)
```

This requires ZERO new MCP infrastructure. Just the BFLs and manifests.

---

## CVar Count Reference (for manifest scoping)

| Plugin  | Approx CVar Count | Key Complexity                              |
|---------|-------------------|---------------------------------------------|
| Vapor | ~55               | Flat structure, some R/G/B triplets         |
| Lucent  | ~100              | 6 Bloom layers × RGB tints, module groups   |
| Tonal   | ~150+             | 8 HueSelect slots × 10 params, VHS Anim    |

Writing the manifests is the most time-consuming part. The BFLs are fast (mostly boilerplate
+ `GetParameterInfo()` hardcoded string).

---

## Non-Goals

- Do NOT build a standalone MCP server (Python/TypeScript)
- Do NOT add HTTP/REST endpoints to the UE plugins
- Do NOT use UE Remote Control plugin (`WebRemoteControl`) — overkill for this use case
- Do NOT add Prometheus/telemetry hooks
- The BFLs do NOT need to be tested in-game — they are editor-only utility bridges

---

## Files Created Per Session

When working on this, track per-plugin file creation:

### Vapor
- `Plugins/Vapor/Source/Vapor/VaporFunctionLibrary.h` — TODO
- `Plugins/Vapor/Source/Vapor/VaporFunctionLibrary.cpp` — TODO
- `Plugins/Vapor/plugindepot-mcp.json` — TODO

### Lucent
- `Plugins/Lucent/Source/Lucent/LucentFunctionLibrary.h` — TODO
- `Plugins/Lucent/Source/Lucent/LucentFunctionLibrary.cpp` — TODO
- `Plugins/Lucent/plugindepot-mcp.json` — TODO

### Tonal
- `Plugins/Tonal/Source/Tonal/TonalFunctionLibrary.h` — TODO
- `Plugins/Tonal/Source/Tonal/TonalFunctionLibrary.cpp` — TODO
- `Plugins/Tonal/plugindepot-mcp.json` — TODO
