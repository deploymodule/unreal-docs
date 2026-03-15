# MCP Integration Todo

## Vapor

### Blueprint Function Library
- [ ] `Plugins/Vapor/Source/Vapor/VaporFunctionLibrary.h`
- [ ] `Plugins/Vapor/Source/Vapor/VaporFunctionLibrary.cpp`
  - `SetEnabled(bool)`
  - `IsEnabled() → bool`
  - `ApplyPreset(int32 PresetIndex)`
  - `GetPresetName(int32) → FString`
  - `SetFloat(FString CVarName, float Value)`
  - `SetBool(FString CVarName, bool Value)`
  - `GetParameterInfo() → FString`  ← hardcoded cheat-sheet string for LLMs
  - `DumpCurrentSettings() → FString`  ← reads live CVar values, returns key=value list

### Manifest
- [ ] `Plugins/Vapor/plugindepot-mcp.json`
  - All ~55 CVars with: name, type, range, default, description
  - Sections: Core, Phase, Chromatic, Color, Artistic, Depth, Fog Sources,
    Spotlight Shafts, Additional Lights, Temporal, Performance, Debug

---

## Lucent

### Blueprint Function Library
- [ ] `Plugins/Lucent/Source/Lucent/LucentFunctionLibrary.h`
- [ ] `Plugins/Lucent/Source/Lucent/LucentFunctionLibrary.cpp`
  - Same base methods as Vapor
  - `SetBloomLayerWeight(int32 Layer, float Weight)`
  - `SetBloomLayerTint(int32 Layer, float R, float G, float B)`
  - `GetParameterInfo() → FString`
  - `DumpCurrentSettings() → FString`

### Manifest
- [ ] `Plugins/Lucent/plugindepot-mcp.json`
  - All ~100 CVars
  - Sections: Global, DOF, Bloom (6 layers), Halation, Streaks, Dirt Mask,
    Glow Pipeline, Distortion, Chromatic Aberration, Misregistration,
    Vignette, Film Grain, Gate Weave

---

## Tonal

### Blueprint Function Library
- [ ] `Plugins/Tonal/Source/Tonal/TonalFunctionLibrary.h`
- [ ] `Plugins/Tonal/Source/Tonal/TonalFunctionLibrary.cpp`
  - Same base methods
  - `ApplyFilmStockPreset(int32)`
  - `SetWhiteBalance(int32 Preset, float CustomTempK)`
  - `SetHueSelectSlot(int32 Slot, float HueCenter, float HueRange, float Strength)`
  - `EnableVHS(bool)`
  - `GetParameterInfo() → FString`
  - `DumpCurrentSettings() → FString`

### Manifest
- [ ] `Plugins/Tonal/plugindepot-mcp.json`
  - All ~150 CVars (including 8-slot HueSelect arrays, VHS Anim params)
  - Sections: Global, Shaper, MicroContrast, Sharpen, SplitTone, SubjectEmphasis,
    BlackFloor, MidtoneGlow, SkinSoftener, HighlightDesat, MidtoneSat, HueShift,
    DepthFade, BleachBypass, FilmStock, HueSelect (8 slots), DepthWB,
    LocalTone, LightWrap, FilmVignette, LGG, VHS, VHS Anim, XPro
  - Film Stock preset table (indices 0–6)
  - White Balance preset table (indices 0–4)

---

## Verification

- [ ] Run `scan_and_index_project_blueprints` after all three BFLs are added
- [ ] Confirm `get_relevant_blueprint_context` returns results for each plugin
- [ ] Confirm `generate_blueprint_logic` can call each BFL's `SetFloat` successfully
