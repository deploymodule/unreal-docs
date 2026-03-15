# Fogline → Vapor Rename Plan

This document tracks every step needed to rename the `Fogline` plugin to `Vapor`.
Work through each phase in order. Check off items as completed.

---

## Phase 0 — Pre-flight

- [ ] Delete `Plugins/Fogline/Binaries/` (stale compiled DLLs)
- [ ] Delete `Plugins/Fogline/Intermediate/` (stale build artifacts)
- [ ] Close Unreal Editor before beginning

---

## Phase 1 — Rename the Top-Level Folder

Using Git (to preserve history):

```bash
git mv "Plugins/Fogline" "Plugins/Vapor"
```

After this, every path below is relative to `Plugins/Vapor/`.

---

## Phase 2 — Rename the .uplugin File

```bash
git mv "Plugins/Vapor/Fogline.uplugin" "Plugins/Vapor/Vapor.uplugin"
```

---

## Phase 3 — Rename Source Module Folders

```bash
git mv "Plugins/Vapor/Source/Fogline"       "Plugins/Vapor/Source/Vapor"
git mv "Plugins/Vapor/Source/FoglineEditor" "Plugins/Vapor/Source/VaporEditor"
```

---

## Phase 4 — Rename Individual Source Files

### Runtime module (`Source/Vapor/`)

```bash
git mv Source/Vapor/Fogline.h                  Source/Vapor/Vapor.h
git mv Source/Vapor/Fogline.cpp                Source/Vapor/Vapor.cpp
git mv Source/Vapor/Fogline.Build.cs           Source/Vapor/Vapor.Build.cs
git mv Source/Vapor/FoglineSettings.h          Source/Vapor/VaporSettings.h
git mv Source/Vapor/FoglineSettings.cpp        Source/Vapor/VaporSettings.cpp
git mv Source/Vapor/FoglineCVars.h             Source/Vapor/VaporCVars.h
git mv Source/Vapor/FoglineCVars.cpp           Source/Vapor/VaporCVars.cpp
git mv Source/Vapor/FoglineVolume.h            Source/Vapor/VaporVolume.h
git mv Source/Vapor/FoglineVolume.cpp          Source/Vapor/VaporVolume.cpp
git mv Source/Vapor/FoglineComponent.h         Source/Vapor/VaporComponent.h
git mv Source/Vapor/FoglineComponent.cpp       Source/Vapor/VaporComponent.cpp
git mv Source/Vapor/FoglineViewExtension.h     Source/Vapor/VaporViewExtension.h
git mv Source/Vapor/FoglineViewExtension.cpp   Source/Vapor/VaporViewExtension.cpp
git mv Source/Vapor/FoglineFunctionLibrary.h   Source/Vapor/VaporFunctionLibrary.h
git mv Source/Vapor/FoglineFunctionLibrary.cpp Source/Vapor/VaporFunctionLibrary.cpp
git mv Source/Vapor/FoglinePreset.h            Source/Vapor/VaporPreset.h
git mv Source/Vapor/FoglinePreset.cpp          Source/Vapor/VaporPreset.cpp
git mv Source/Vapor/FoglineHeightFog.h         Source/Vapor/VaporHeightFog.h
git mv Source/Vapor/FoglineHeightFog.cpp       Source/Vapor/VaporHeightFog.cpp
git mv Source/Vapor/FoglineShaftLights.h       Source/Vapor/VaporShaftLights.h
git mv Source/Vapor/FoglineShaftLights.cpp     Source/Vapor/VaporShaftLights.cpp
git mv Source/Vapor/FoglineVolumeSubsystem.h   Source/Vapor/VaporVolumeSubsystem.h
git mv Source/Vapor/FoglineVolumeSubsystem.cpp Source/Vapor/VaporVolumeSubsystem.cpp
```

### Editor module (`Source/VaporEditor/`)

```bash
git mv Source/VaporEditor/FoglineEditor.Build.cs  Source/VaporEditor/VaporEditor.Build.cs
git mv Source/VaporEditor/FoglineEditor.h         Source/VaporEditor/VaporEditor.h
git mv Source/VaporEditor/FoglineEditor.cpp       Source/VaporEditor/VaporEditor.cpp
git mv Source/VaporEditor/FoglinePanel.h          Source/VaporEditor/VaporPanel.h
git mv Source/VaporEditor/FoglinePanel.cpp        Source/VaporEditor/VaporPanel.cpp
```

### Sequencer classes (`Source/VaporEditor/Sequencer/`)

```bash
git mv Source/VaporEditor/Sequencer/FoglineTrackEditor.h                   Source/VaporEditor/Sequencer/VaporTrackEditor.h
git mv Source/VaporEditor/Sequencer/FoglineTrackEditor.cpp                 Source/VaporEditor/Sequencer/VaporTrackEditor.cpp
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineTrack.h               Source/VaporEditor/Sequencer/MovieSceneVaporTrack.h
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineTrack.cpp             Source/VaporEditor/Sequencer/MovieSceneVaporTrack.cpp
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineSection.h             Source/VaporEditor/Sequencer/MovieSceneVaporSection.h
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineSection.cpp           Source/VaporEditor/Sequencer/MovieSceneVaporSection.cpp
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineSectionTemplate.h     Source/VaporEditor/Sequencer/MovieSceneVaporSectionTemplate.h
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineSectionTemplate.cpp   Source/VaporEditor/Sequencer/MovieSceneVaporSectionTemplate.cpp
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeTrack.h         Source/VaporEditor/Sequencer/MovieSceneVaporVolumeTrack.h
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeTrack.cpp       Source/VaporEditor/Sequencer/MovieSceneVaporVolumeTrack.cpp
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeSection.h       Source/VaporEditor/Sequencer/MovieSceneVaporVolumeSection.h
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeSection.cpp     Source/VaporEditor/Sequencer/MovieSceneVaporVolumeSection.cpp
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeTemplate.h      Source/VaporEditor/Sequencer/MovieSceneVaporVolumeTemplate.h
git mv Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeTemplate.cpp    Source/VaporEditor/Sequencer/MovieSceneVaporVolumeTemplate.cpp
```

### Shader files (`Shaders/`)

Rename all shader files that reference Fogline in their name:

```bash
git mv Shaders/FoglineSetup.usf      Shaders/VaporSetup.usf
git mv Shaders/FoglineDownsample.usf Shaders/VaporDownsample.usf
git mv Shaders/FoglineUpsample.usf   Shaders/VaporUpsample.usf
git mv Shaders/FoglineRecombine.usf  Shaders/VaporRecombine.usf
git mv Shaders/FoglineCommon.ush     Shaders/VaporCommon.ush
git mv Shaders/FoglineHeightFog.usf  Shaders/VaporHeightFog.usf
```

---

## Phase 5 — Find-Replace Inside All Files

After renaming files/folders, every file with "Fogline" content must be updated.
Run these substitutions across all files in `Plugins/Vapor/` (and the MCP manifest):

### 5a — Ordered substitutions (apply IN THIS ORDER to avoid double-replacing)

| Find (exact) | Replace with | Notes |
|---|---|---|
| `FOGLINE_API` | `VAPOR_API` | DLL export macro |
| `FOGLINE_MAX_ADDITIONAL_LIGHTS` | `VAPOR_MAX_ADDITIONAL_LIGHTS` | Constant |
| `LogFogline` | `LogVapor` | Log category |
| `r.Fogline` | `r.Vapor` | CVar prefix (handles all 60 CVars) |
| `FFoglineModule` | `FVaporModule` | Module class |
| `FFoglineSettings` | `FVaporSettings` | Settings struct |
| `FFoglineViewExtension` | `FVaporViewExtension` | View extension class |
| `FFoglineAdditionalLightParameters` | `FVaporAdditionalLightParameters` | Shader struct |
| `AFoglineVolume` | `AVaporVolume` | Volume actor |
| `UFoglineComponent` | `UVaporComponent` | Component |
| `UFoglinePreset` | `UVaporPreset` | Data asset |
| `UFoglineFunctionLibrary` | `UVaporFunctionLibrary` | BFL |
| `UFoglineVolumeSubsystem` | `UVaporVolumeSubsystem` | Subsystem |
| `UMovieSceneFoglineTrack` | `UMovieSceneVaporTrack` | Sequencer |
| `UMovieSceneFoglineSection` | `UMovieSceneVaporSection` | Sequencer |
| `UMovieSceneFoglineVolumeTrack` | `UMovieSceneVaporVolumeTrack` | Sequencer |
| `UMovieSceneFoglineVolumeSection` | `UMovieSceneVaporVolumeSection` | Sequencer |
| `UFoglineVolumeTrackInstance` | `UVaporVolumeTrackInstance` | Sequencer |
| `UFoglineTrackInstance` | `UVaporTrackInstance` | Sequencer |
| `FFoglineTrackEditor` | `FVaporTrackEditor` | Sequencer editor |
| `FFoglineSettingsKey` | `FVaporSettingsKey` | Sequencer key |
| `FFoglineVolumeKey` | `FVaporVolumeKey` | Sequencer key |
| `FoglineEditor` | `VaporEditor` | Module name (Build.cs, IMPLEMENT_MODULE, includes) |
| `class Fogline ` | `class Vapor ` | Build.cs class declaration |
| `public Fogline(` | `public Vapor(` | Build.cs constructor |
| `#include "Fogline` | `#include "Vapor` | All include paths |
| `"Fogline"` | `"Vapor"` | String literals (plugin name, module lookup) |
| `TEXT("Fogline")` | `TEXT("Vapor")` | UE text macros |
| `Category = "Fogline` | `Category = "Vapor` | UPROPERTY/UFUNCTION categories |
| `ClassGroup = (Fogline)` | `ClassGroup = (Vapor)` | UCLASS metadata |
| `IMPLEMENT_MODULE(FVaporModule, Fogline)` | `IMPLEMENT_MODULE(FVaporModule, Vapor)` | Module registration (fix after prior steps) |
| `FoglineViewExtension` | `VaporViewExtension` | Any remaining bare references |
| `FoglineSettings` | `VaporSettings` | Any remaining bare references |
| `FoglineVolume` | `VaporVolume` | Any remaining bare references |
| `FoglineComponent` | `VaporComponent` | Any remaining bare references |
| `FoglinePanel` | `VaporPanel` | Any remaining bare references |
| `FoglineTrackEditor` | `VaporTrackEditor` | Any remaining bare references |

### 5b — Shader file includes (inside .usf/.ush)

Inside the renamed shader files, update any `#include "Fogline*.ush"` paths to `#include "Vapor*.ush"`.

---

## Phase 6 — Update Vapor.uplugin

Edit `Plugins/Vapor/Vapor.uplugin`:

```json
{
  "FriendlyName": "Vapor",
  "Description": "Enhanced fog light scattering via compute-shader-driven downsample/upsample pipeline",
  "Modules": [
    { "Name": "Vapor",       "Type": "Runtime",  ... },
    { "Name": "VaporEditor", "Type": "Editor",   ... }
  ]
}
```

- Change `"FriendlyName": "Fogline"` → `"FriendlyName": "Vapor"`
- Change module `"Name": "Fogline"` → `"Name": "Vapor"`
- Change module `"Name": "FoglineEditor"` → `"Name": "VaporEditor"`

---

## Phase 7 — Update MCP Manifest

Edit `Plugins/Vapor/plugindepot-mcp.json`:

- `"plugin": "Fogline"` → `"plugin": "Vapor"`
- `"blueprint_function_library": "UFoglineFunctionLibrary"` → `"blueprint_function_library": "UVaporFunctionLibrary"`
- All 60 CVar names from `r.Fogline.*` → `r.Vapor.*` (covered by Phase 5a row `r.Fogline` → `r.Vapor`)

---

## Phase 8 — Check Project .uproject

Open `PluginDepot.uproject` and search for any `"Fogline"` entries in the Plugins array.
Replace `"Name": "Fogline"` with `"Name": "Vapor"`.

---

## Phase 9 — Check Other Plugins / Docs for Cross-References

Search the whole repo for remaining "Fogline" text outside `Plugins/Vapor/`:

```bash
grep -ri "fogline" . --include="*.h" --include="*.cpp" --include="*.cs" \
     --include="*.usf" --include="*.ush" --include="*.json" \
     --include="*.md" --include="*.uplugin" --include="*.uproject" \
     --exclude-dir=Vapor
```

Known locations that may reference Fogline:
- `docs/marrow-todo.md` — check for any cross-plugin mentions
- `CLAUDE.md` — update Fogline section heading to Vapor
- `memory/MEMORY.md` — update "Fogline" plugin entry to "Vapor"
- `Content/FastTests/` — check for any Fogline test assets

---

## Phase 10 — Generated File Cleanup

After all source changes, delete any leftover generated files that embed the old name.
These will be regenerated on the next build:

- `Plugins/Vapor/Source/Vapor/FoglineSettings.generated.h` (if not renamed by git mv)
- Any `.generated.h` files still containing "Fogline"

---

## Phase 11 — Rebuild Verification

1. Right-click `PluginDepot.uproject` → **Generate Visual Studio Project Files**
2. Build the project (Development Editor)
3. Confirm zero compiler errors referencing `Fogline`
4. Open Unreal Editor and verify:
   - Vapor plugin appears in Plugins list (Edit → Plugins)
   - `r.Vapor.*` CVars appear in console (type `r.Vapor` and tab-complete)
   - `UFoglineFunctionLibrary` is gone; `UVaporFunctionLibrary` appears in Blueprints
   - Sequencer: adding a Vapor track works on a Volume actor

---

## Phase 12 — Git Commit

```bash
git add -A
git commit -m "Rename Fogline plugin to Vapor"
```

---

## Quick Reference: All File Renames

| Old path (relative to Plugins/Fogline → Plugins/Vapor) | New name |
|---|---|
| `Fogline.uplugin` | `Vapor.uplugin` |
| `Source/Fogline/` (folder) | `Source/Vapor/` |
| `Source/FoglineEditor/` (folder) | `Source/VaporEditor/` |
| `Source/Vapor/Fogline.h` | `Vapor.h` |
| `Source/Vapor/Fogline.cpp` | `Vapor.cpp` |
| `Source/Vapor/Fogline.Build.cs` | `Vapor.Build.cs` |
| `Source/Vapor/FoglineSettings.h` | `VaporSettings.h` |
| `Source/Vapor/FoglineSettings.cpp` | `VaporSettings.cpp` |
| `Source/Vapor/FoglineCVars.h` | `VaporCVars.h` |
| `Source/Vapor/FoglineCVars.cpp` | `VaporCVars.cpp` |
| `Source/Vapor/FoglineVolume.h` | `VaporVolume.h` |
| `Source/Vapor/FoglineVolume.cpp` | `VaporVolume.cpp` |
| `Source/Vapor/FoglineComponent.h` | `VaporComponent.h` |
| `Source/Vapor/FoglineComponent.cpp` | `VaporComponent.cpp` |
| `Source/Vapor/FoglineViewExtension.h` | `VaporViewExtension.h` |
| `Source/Vapor/FoglineViewExtension.cpp` | `VaporViewExtension.cpp` |
| `Source/Vapor/FoglineFunctionLibrary.h` | `VaporFunctionLibrary.h` |
| `Source/Vapor/FoglineFunctionLibrary.cpp` | `VaporFunctionLibrary.cpp` |
| `Source/Vapor/FoglinePreset.h` | `VaporPreset.h` |
| `Source/Vapor/FoglinePreset.cpp` | `VaporPreset.cpp` |
| `Source/Vapor/FoglineHeightFog.h` | `VaporHeightFog.h` |
| `Source/Vapor/FoglineHeightFog.cpp` | `VaporHeightFog.cpp` |
| `Source/Vapor/FoglineShaftLights.h` | `VaporShaftLights.h` |
| `Source/Vapor/FoglineShaftLights.cpp` | `VaporShaftLights.cpp` |
| `Source/Vapor/FoglineVolumeSubsystem.h` | `VaporVolumeSubsystem.h` |
| `Source/Vapor/FoglineVolumeSubsystem.cpp` | `VaporVolumeSubsystem.cpp` |
| `Source/VaporEditor/FoglineEditor.Build.cs` | `VaporEditor.Build.cs` |
| `Source/VaporEditor/FoglineEditor.h` | `VaporEditor.h` |
| `Source/VaporEditor/FoglineEditor.cpp` | `VaporEditor.cpp` |
| `Source/VaporEditor/FoglinePanel.h` | `VaporPanel.h` |
| `Source/VaporEditor/FoglinePanel.cpp` | `VaporPanel.cpp` |
| `Source/VaporEditor/Sequencer/FoglineTrackEditor.h` | `VaporTrackEditor.h` |
| `Source/VaporEditor/Sequencer/FoglineTrackEditor.cpp` | `VaporTrackEditor.cpp` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineTrack.h` | `MovieSceneVaporTrack.h` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineTrack.cpp` | `MovieSceneVaporTrack.cpp` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineSection.h` | `MovieSceneVaporSection.h` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineSection.cpp` | `MovieSceneVaporSection.cpp` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineSectionTemplate.h` | `MovieSceneVaporSectionTemplate.h` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineSectionTemplate.cpp` | `MovieSceneVaporSectionTemplate.cpp` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeTrack.h` | `MovieSceneVaporVolumeTrack.h` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeTrack.cpp` | `MovieSceneVaporVolumeTrack.cpp` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeSection.h` | `MovieSceneVaporVolumeSection.h` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeSection.cpp` | `MovieSceneVaporVolumeSection.cpp` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeTemplate.h` | `MovieSceneVaporVolumeTemplate.h` |
| `Source/VaporEditor/Sequencer/MovieSceneFoglineVolumeTemplate.cpp` | `MovieSceneVaporVolumeTemplate.cpp` |
| `Shaders/FoglineSetup.usf` | `VaporSetup.usf` |
| `Shaders/FoglineDownsample.usf` | `VaporDownsample.usf` |
| `Shaders/FoglineUpsample.usf` | `VaporUpsample.usf` |
| `Shaders/FoglineRecombine.usf` | `VaporRecombine.usf` |
| `Shaders/FoglineCommon.ush` | `VaporCommon.ush` |
| `Shaders/FoglineHeightFog.usf` | `VaporHeightFog.usf` |

---

## Quick Reference: All Symbol Renames

| Old | New |
|---|---|
| `FOGLINE_API` | `VAPOR_API` |
| `FOGLINE_MAX_ADDITIONAL_LIGHTS` | `VAPOR_MAX_ADDITIONAL_LIGHTS` |
| `LogFogline` | `LogVapor` |
| `r.Fogline.*` (60 CVars) | `r.Vapor.*` |
| `FFoglineModule` | `FVaporModule` |
| `FFoglineSettings` | `FVaporSettings` |
| `FFoglineViewExtension` | `FVaporViewExtension` |
| `FFoglineAdditionalLightParameters` | `FVaporAdditionalLightParameters` |
| `AFoglineVolume` | `AVaporVolume` |
| `UFoglineComponent` | `UVaporComponent` |
| `UFoglinePreset` | `UVaporPreset` |
| `UFoglineFunctionLibrary` | `UVaporFunctionLibrary` |
| `UFoglineVolumeSubsystem` | `UVaporVolumeSubsystem` |
| `UMovieSceneFoglineTrack` | `UMovieSceneVaporTrack` |
| `UMovieSceneFoglineSection` | `UMovieSceneVaporSection` |
| `UFoglineTrackInstance` | `UVaporTrackInstance` |
| `UMovieSceneFoglineVolumeTrack` | `UMovieSceneVaporVolumeTrack` |
| `UMovieSceneFoglineVolumeSection` | `UMovieSceneVaporVolumeSection` |
| `UFoglineVolumeTrackInstance` | `UVaporVolumeTrackInstance` |
| `FFoglineTrackEditor` | `FVaporTrackEditor` |
| `FFoglineSettingsKey` | `FVaporSettingsKey` |
| `FFoglineVolumeKey` | `FVaporVolumeKey` |
| `IMPLEMENT_MODULE(..., Fogline)` | `IMPLEMENT_MODULE(..., Vapor)` |
| Module name `"Fogline"` | `"Vapor"` |
| Module name `"FoglineEditor"` | `"VaporEditor"` |
