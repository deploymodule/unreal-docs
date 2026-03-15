# Lucent: Animation, Sequencer & Runtime Control — Implementation Plan

## Context

Lucent is an Unreal Engine 5.7 cinematic lens and film stack plugin (compute shader pipeline via
`FSceneViewExtensionBase`). Parameters live as CVars at render time; `FLucentSettings` is a
`USTRUCT(BlueprintType)` that mirrors CVars for game-thread access. `ULucentComponent` is a
`BlueprintSpawnableComponent` that owns `FLucentSettings` and pushes it to CVars via
`bAutoApply` tick or explicit `ApplySettings()`. `ULucentPreset` is a `UDataAsset` that stores
a named `FLucentSettings` snapshot.

This document is the Lucent port of the same system built for Tonal. For design patterns,
reference `Plugins/Tonal/` as the completed implementation.

## Goal

Full support for:
- **Sequencer animation** — keyframe any parameter in the editor timeline, with working viewport preview
- **Blueprint control** — every parameter reachable via BP nodes
- **C++ runtime control** — structured API on top of the CVar layer
- **Smooth transitions** — lerp between named presets or arbitrary snapshots at runtime

---

## Todo List

### Phase 1 — Close the FLucentSettings CVar Gap

**Status: Complete**

`FLucentSettings` covers ~97% of the CVars but three parameters exist in the CVar layer and
`FFrameData` that are absent from the settings struct, making them unreachable from Blueprint
and Sequencer. Add each to:
1. `FLucentSettings` as `UPROPERTY(EditAnywhere, BlueprintReadWrite, ...)` fields
2. `FLucentSettings::ApplyToCVars()` — push to CVars
3. `FLucentSettings::ReadFromCVars()` — pull from CVars
4. `FLucentSettings::Lerp()` — interpolate

#### 1.1 Distortion — Custom Coefficients Toggle

The CVar `r.Lucent.Distortion.CustomCoefficients` selects between artist-friendly barrel
amount (0) and raw lens calibration coefficients (1). Without this in the struct, Blueprint
users cannot switch modes.

Field to add in the Distortion section, after `DistortionBarrelAmount`:

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Lucent|Distortion",
    meta = (EditCondition = "bDistortionEnabled",
            ToolTip = "When true, uses raw K1/K2/K3/P1/P2 coefficients instead of the artist-friendly Barrel Amount."))
bool bDistortionUseRawCoefficients = false;
```

`ApplyToCVars()` addition:
```cpp
CVarLucentDistortionCustomCoefficients->Set(bDistortionUseRawCoefficients ? 1 : 0, ECVF_SetByConsole);
```

`Lerp()`: snaps at Alpha >= 0.5.

---

#### 1.2 Distortion — Post-Distortion Sharpening

The CVar `r.Lucent.Distortion.Sharpening` applies unsharp-mask after the warp pass. Without
this in the struct it cannot be keyframed.

Field to add after `DistortionScale`:

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Lucent|Distortion",
    meta = (ClampMin = "0.0", ClampMax = "0.5", EditCondition = "bDistortionEnabled",
            ToolTip = "Post-distortion unsharp-mask sharpening. Compensates perceived softness after barrel warp."))
float DistortionSharpening = 0.0f;
```

`ApplyToCVars()` addition:
```cpp
CVarLucentDistortionSharpening->Set(DistortionSharpening, ECVF_SetByConsole);
```

`Lerp()`: lerps linearly.

---

#### 1.3 Chromatic Aberration — Feather

The CVar `r.Lucent.CA.Feather` controls the falloff transition width of the CA-free inner
radius zone. Without this, the CA edge blend cannot be animated.

Field to add in the CA section after `CAInnerRadius`:

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Lucent|Chromatic Aberration",
    meta = (ClampMin = "0.0", ClampMax = "1.0", EditCondition = "bCAEnabled",
            ToolTip = "Transition width from CA-free inner zone to full CA outer region."))
float CAFeather = 0.5f;
```

`ApplyToCVars()` addition:
```cpp
CVarLucentCAFeather->Set(CAFeather, ECVF_SetByConsole);
```

`Lerp()`: lerps linearly.

---

### Phase 2 — Enable Editor-Tick for Sequencer Preview

**Status: Complete**

#### 2.1 Mark `ULucentComponent` to tick in editor

In `LucentComponent.h`, change the class specifier:

```cpp
UCLASS(Blueprintable, ClassGroup = (Rendering),
    meta = (BlueprintSpawnableComponent, bTickInEditor = true),
    HideCategories = (Collision, Physics, Navigation))
class LUCENT_API ULucentComponent : public UActorComponent
```

In the constructor in `LucentComponent.cpp`, ensure tick is initialized for editor use:

```cpp
PrimaryComponentTick.bCanEverTick = true;
PrimaryComponentTick.bStartWithTickEnabled = false;
PrimaryComponentTick.bTickEvenWhenPaused = false;
```

#### 2.2 Sequencer scrubbing: verify editor tick drives CVars

When `bAutoApply` is true and `bTickInEditor` is set, Sequencer scrubbing in the editor will
tick the component, calling `ApplySettings()` → `ApplyToCVars()` → GPU sees updated values.
Verify this works by testing Sequencer preview in a non-PIE viewport.

---

### Phase 3 — Expand `ULucentComponent` Blueprint API

**Status: Complete**

#### 3.1 Add convenience setters for Phase 1 parameters and common controls

Follow the same pattern as the existing `SetBloomIntensity` / `SetVignetteIntensity` setters.
Each setter updates the corresponding `Settings` field AND pushes directly to the CVar.

Setters to add (minimum required set):

```cpp
// Distortion
UFUNCTION(BlueprintCallable, Category = "Lucent|Distortion")
void SetDistortionBarrelAmount(float Value);

UFUNCTION(BlueprintCallable, Category = "Lucent|Distortion")
void SetDistortionSharpening(float Value);

// DOF
UFUNCTION(BlueprintCallable, Category = "Lucent|DOF")
void SetDOFEnabled(bool bEnabled);

UFUNCTION(BlueprintCallable, Category = "Lucent|DOF")
void SetDOFMaxRadius(float Value);

UFUNCTION(BlueprintCallable, Category = "Lucent|DOF")
void SetDOFSwirl(float Value);

// Bloom
UFUNCTION(BlueprintCallable, Category = "Lucent|Bloom")
void SetBloomEnabled(bool bEnabled);

UFUNCTION(BlueprintCallable, Category = "Lucent|Bloom")
void SetBloomThreshold(float Value);

// Halation
UFUNCTION(BlueprintCallable, Category = "Lucent|Halation")
void SetHalationEnabled(bool bEnabled);

UFUNCTION(BlueprintCallable, Category = "Lucent|Halation")
void SetHalationHue(float Value);

// Streaks
UFUNCTION(BlueprintCallable, Category = "Lucent|Streaks")
void SetStreaksEnabled(bool bEnabled);

UFUNCTION(BlueprintCallable, Category = "Lucent|Streaks")
void SetStreaksCount(int32 Count);

// Vignette
UFUNCTION(BlueprintCallable, Category = "Lucent|Vignette")
void SetVignetteEnabled(bool bEnabled);

// Chromatic Aberration
UFUNCTION(BlueprintCallable, Category = "Lucent|Chromatic Aberration")
void SetCAEnabled(bool bEnabled);

// Grain
UFUNCTION(BlueprintCallable, Category = "Lucent|Film Grain")
void SetGrainEnabled(bool bEnabled);

// Weave
UFUNCTION(BlueprintCallable, Category = "Lucent|Gate Weave")
void SetWeaveEnabled(bool bEnabled);

// Global
UFUNCTION(BlueprintCallable, Category = "Lucent")
void SetQuality(int32 InQuality);
```

#### 3.2 Add `SnapshotFromCurrent()` helper

```cpp
/** Capture current CVar state into this component's Settings without applying.
    Useful for setting up a Lerp baseline from whatever is currently live. */
UFUNCTION(BlueprintCallable, Category = "Lucent")
void SnapshotFromCurrent();
```

Implementation: calls `Settings.ReadFromCVars()`.

#### 3.3 Add `LerpToSettingsCurved()` optional quality upgrade

```cpp
/** Lerp to target settings with a curve-remapped alpha for easing. */
UFUNCTION(BlueprintCallable, Category = "Lucent")
void LerpToSettingsCurved(const FLucentSettings& Target, float Alpha, UCurveFloat* Curve);
```

Evaluates `Curve->GetFloatValue(Alpha)` to remap the interpolation shape (ease-in/ease-out)
before passing to the existing `FLucentSettings::Lerp()`.

---

### Phase 4 — Verify `Lerp()` Completeness

**Status: Complete** — audited during Phase 1 implementation. All 3 new fields wired in. See Phase 1 notes.

After Phase 1 adds new fields to `FLucentSettings`, audit `FLucentSettings::Lerp()` in
`LucentSettings.cpp` to confirm:
1. Every field in the struct has a corresponding line in `Lerp()`
2. `TSoftObjectPtr<UTexture2D>` fields snap at Alpha >= 0.5 (cannot meaningfully interpolate asset refs)
3. All `ELucentOperationMode` enum fields snap at Alpha >= 0.5
4. All `ELucentDebugMode` fields snap at Alpha >= 0.5
5. All float fields lerp linearly
6. All bool and int32 fields snap at Alpha >= 0.5

Checklist per module:
- [ ] Global — Quality (snap), bMRQMode (snap), DebugMode (snap)
- [ ] DOF — all floats lerp; bDOFEnabled, DOFMode, DOFBladeCount, DOFRingCount snap; DOFKernelTexture snaps
- [ ] Bloom — all floats lerp; bBloomEnabled, BloomMode snap; BloomLayer[0-5]Tint lerps per-channel (R/G/B)
- [ ] Halation — all floats lerp; bHalationEnabled, HalationMipLevel snap
- [ ] Streaks — all floats lerp; bStreaksEnabled, StreaksCount snap; StreaksTint lerps per-channel
- [ ] Dirt — DirtIntensity lerps; bDirtEnabled, DirtMode snap; DirtTint lerps per-channel; DirtMaskTexture snaps
- [ ] Glow Pipeline — GlowDownsample snaps; GlowLevels snaps; GlowThreshold, GlowThresholdKnee, GlowBlurSigma lerp
- [ ] Distortion — all floats lerp; bDistortionEnabled, DistortionMode, bDistortionUseRawCoefficients snap
- [ ] CA — all floats lerp; bCAEnabled, CAMode, CASamples snap
- [ ] Misregistration — all floats lerp; bMisregEnabled, MisregChannel, bMisregJitter snap
- [ ] Vignette — all floats lerp; bVignetteEnabled, VignetteMode snap
- [ ] Film Grain — all floats lerp; bGrainEnabled, GrainMode snap
- [ ] Gate Weave — all floats lerp; bWeaveEnabled snaps
- [ ] Performance — bPreserveAlpha snaps

---

### Phase 5 — Custom Sequencer Track

**Status: Not started** (Phases 1–4 and Build.cs prerequisite from 5.6 are complete)

This is the primary Sequencer integration. It gives artists a dedicated "Lucent Lens" track in
the Sequencer timeline, frame-accurate evaluation (no tick dependency), and in-editor viewport
preview while scrubbing. It does not require `bAutoApply` to be enabled on any component.

#### Design: Snapshot-Key Model

The track uses a **snapshot-per-key** data model rather than per-channel float curves.
Each key stores a complete `FLucentSettings` value. At evaluation time the two bracketing keys
are found and `FLucentSettings::Lerp()` interpolates between them. Artists think in whole looks
("Exterior Day → Interior Night"), not individual float curves.

---

#### 5.1 New key struct — `Source/LucentEditor/Sequencer/LucentSettingsKey.h`

```cpp
#pragma once
#include "LucentSettings.h"
#include "LucentSettingsKey.generated.h"

/** A single timed lens keyframe for the Lucent Sequencer track. */
USTRUCT()
struct FLucentSettingsKey
{
    GENERATED_BODY()

    UPROPERTY()
    FFrameNumber Time;

    UPROPERTY()
    FLucentSettings Value;
};
```

---

#### 5.2 Section — `Source/LucentEditor/Sequencer/MovieSceneLucentSection.h/.cpp`

```cpp
UCLASS()
class ULucentSection : public UMovieSceneSection
{
    GENERATED_BODY()
public:
    ULucentSection();

    /** Keys sorted ascending by Time. */
    UPROPERTY()
    TArray<FLucentSettingsKey> Keys;

    /** Add or overwrite a key at the given frame. */
    void AddKey(FFrameNumber Time, const FLucentSettings& InValue);

    /** Evaluate the interpolated lens settings at the given time.
        Returns default settings if the section has no keys. */
    FLucentSettings Evaluate(FFrameTime Time) const;
};
```

`Evaluate()` logic:
- Empty → return `FLucentSettings{}`
- One key, or time ≤ first key → return `Keys[0].Value`
- Time ≥ last key → return `Keys.Last().Value`
- Otherwise: find bracketing pair `[i]` / `[i+1]`, compute
  `Alpha = (Time.AsDecimal() - Keys[i].Time.Value) / float(Keys[i+1].Time.Value - Keys[i].Time.Value)`,
  return `FLucentSettings::Lerp(Keys[i].Value, Keys[i+1].Value, Alpha)`

---

#### 5.3 Track — `Source/LucentEditor/Sequencer/MovieSceneLucentTrack.h/.cpp`

```cpp
UCLASS(MinimalAPI, BlueprintType)
class UMovieSceneLucentTrack : public UMovieSceneNameableTrack
{
    GENERATED_BODY()
public:
    virtual UMovieSceneSection* CreateNewSection() override;
    virtual void AddSection(UMovieSceneSection& Section) override;
    virtual bool HasSection(const UMovieSceneSection& Section) const override;
    virtual void RemoveSection(UMovieSceneSection& Section) override;
    virtual void RemoveSectionAt(int32 SectionIndex) override;
    virtual bool IsEmpty() const override;
    virtual const TArray<UMovieSceneSection*>& GetAllSections() const override;
    virtual bool SupportsType(TSubclassOf<UMovieSceneSection> SectionClass) const override;
    virtual FMovieSceneEvalTemplatePtr CreateTemplateForSection(
        const UMovieSceneSection& InSection) const override;

#if WITH_EDITORONLY_DATA
    virtual FText GetDisplayName() const override
    { return NSLOCTEXT("Lucent", "TrackName", "Lucent Lens"); }
#endif

private:
    UPROPERTY()
    TArray<TObjectPtr<UMovieSceneSection>> Sections;
};
```

`CreateNewSection()` allocates a `ULucentSection` and sets its range to the owning sequence's
full playback range.

`CreateTemplateForSection()` returns a `FLucentSectionTemplate` (5.4).

---

#### 5.4 Evaluation template — `Source/LucentEditor/Sequencer/MovieSceneLucentSectionTemplate.h/.cpp`

```cpp
// Execution token — runs on game thread, applies evaluated settings to CVars.
struct FLucentEvalToken : IMovieSceneExecutionToken
{
    FLucentSettings Settings;

    virtual void Execute(const FMovieSceneContext&,
                         const FMovieSceneEvaluationOperand&,
                         FPersistentEvaluationData&,
                         IMovieScenePlayer&) override
    {
        Settings.ApplyToCVars();
    }
};

// Template — called by the Sequencer evaluation graph once per section per frame.
USTRUCT()
struct FLucentSectionTemplate : FMovieSceneEvalTemplate
{
    GENERATED_BODY()

    FLucentSectionTemplate() = default;
    explicit FLucentSectionTemplate(const ULucentSection& Section);

    UPROPERTY()
    TWeakObjectPtr<const ULucentSection> SectionPtr;

    virtual UScriptStruct& GetScriptStructImpl() const override { return *StaticStruct(); }

    virtual void Evaluate(const FMovieSceneEvaluationOperand&,
                          const FMovieSceneContext& Context,
                          const FPersistentEvaluationData&,
                          FMovieSceneExecutionTokens& ExecutionTokens) const override
    {
        if (const ULucentSection* Section = SectionPtr.Get())
        {
            FLucentEvalToken Token;
            Token.Settings = Section->Evaluate(Context.GetTime());
            ExecutionTokens.Add(MoveTemp(Token));
        }
    }
};
```

The base header in UE5.7 is `MovieSceneEvalTemplate.h` under `MovieScene/Public/Evaluation/`.
Reference `FMovieSceneCameraShakeSectionTemplate` as a real-engine pattern for this design.

---

#### 5.5 Track editor — `Source/LucentEditor/Sequencer/LucentTrackEditor.h/.cpp`

```cpp
class FLucentTrackEditor : public FMovieSceneTrackEditor
{
public:
    static TSharedRef<ISequencerTrackEditor> CreateTrackEditor(TSharedRef<ISequencer> Sequencer);
    explicit FLucentTrackEditor(TSharedRef<ISequencer> InSequencer);

    virtual bool SupportsType(TSubclassOf<UMovieSceneTrack> TrackClass) const override;
    virtual void BuildAddTrackMenu(FMenuBuilder& MenuBuilder) override;
    virtual bool HandleAssetAdded(UObject* Asset, const FGuid& TargetObjectGuid) override;

private:
    void AddLucentTrack();
    void AddKeyAtCurrentTime(ULucentSection* Section);
};
```

Behaviours:
- `BuildAddTrackMenu`: adds "Lucent Lens" under the master tracks section; calls `AddLucentTrack()`.
  The track is a **global master track** (add via `UMovieScene::AddMasterTrack<UMovieSceneLucentTrack>()`).
  No object binding needed — Lucent is a global post-process effect.
- `HandleAssetAdded`: if a `ULucentPreset` is dragged onto the timeline, create a key at the
  current time with `Preset->Settings` as the value.
- `AddKeyAtCurrentTime`: calls `FLucentSettings::ReadFromCVars()` to snapshot the current live
  state, then calls `ULucentSection::AddKey()`.

---

#### 5.6 Module dependencies — `LucentEditor.Build.cs`

**Applied 2026-02-27.** Already present in `PrivateDependencyModuleNames`:

```csharp
"MovieScene",
"MovieSceneTracks",
"Sequencer",
"SequencerCore",
```

---

#### 5.7 Registration in `FLucentEditorModule`

In `LucentEditor.h`, add a private member:

```cpp
FDelegateHandle TrackEditorHandle;
```

In `FLucentEditorModule::StartupModule()`, after the existing tab setup:

```cpp
ISequencerModule& SequencerModule =
    FModuleManager::LoadModuleChecked<ISequencerModule>("Sequencer");
TrackEditorHandle = SequencerModule.RegisterTrackEditor(
    FOnCreateTrackEditor::CreateStatic(&FLucentTrackEditor::CreateTrackEditor));
```

In `FLucentEditorModule::ShutdownModule()`:

```cpp
if (ISequencerModule* SequencerModule =
    FModuleManager::GetModulePtr<ISequencerModule>("Sequencer"))
{
    SequencerModule->UnRegisterTrackEditor(TrackEditorHandle);
}
```

---

## Implementation Order

1. **Phase 1** — Add the three missing fields to `FLucentSettings`. Do all three (1.1–1.3) in
   one pass through `LucentSettings.h` and `LucentSettings.cpp`. This is the prerequisite for
   a complete Lerp().

2. **Phase 4** — Immediately after Phase 1, audit `Lerp()`. Do this while the new fields are
   fresh.

3. **Phase 2** — One-line `bTickInEditor` change. Do alongside Phase 1.

4. **Phase 3** — Convenience setters + `SnapshotFromCurrent()`. Do after Phase 1 compiles cleanly.

5. **Phase 5** — Build the Sequencer track. Phases 1 and 4 must be complete first so
   `FLucentSettings::Lerp()` is correct before the section's `Evaluate()` calls it.
   Work in order: 5.1 → 5.2 → 5.3 → 5.4 → 5.5 → 5.6 → 5.7.

---

## Key Files

| File | Role |
|---|---|
| `Source/Lucent/LucentSettings.h` | Add 3 missing UPROPERTY fields (Phase 1) |
| `Source/Lucent/LucentSettings.cpp` | `ApplyToCVars`, `ReadFromCVars`, `Lerp` additions and audit (Phases 1, 4) |
| `Source/Lucent/LucentCVars.h` | Verify CVarLucentDistortionCustomCoefficients, CVarLucentDistortionSharpening, CVarLucentCAFeather are declared |
| `Source/Lucent/LucentComponent.h` | `bTickInEditor=true`, new UFUNCTION setters, `SnapshotFromCurrent`, `LerpToSettingsCurved` (Phases 2, 3) |
| `Source/Lucent/LucentComponent.cpp` | Implement new setters (Phase 3) |
| `Source/LucentEditor/LucentEditor.h` | Add `TrackEditorHandle` field (Phase 5.7) |
| `Source/LucentEditor/LucentEditor.cpp` | Register/unregister track editor (Phase 5.7) |
| `Source/LucentEditor/LucentEditor.Build.cs` | Add MovieScene/Sequencer deps (Phase 5.6) |
| `Source/LucentEditor/Sequencer/LucentSettingsKey.h` | Key struct (Phase 5.1) |
| `Source/LucentEditor/Sequencer/MovieSceneLucentSection.h/.cpp` | Section + Evaluate() (Phase 5.2) |
| `Source/LucentEditor/Sequencer/MovieSceneLucentTrack.h/.cpp` | Track asset (Phase 5.3) |
| `Source/LucentEditor/Sequencer/MovieSceneLucentSectionTemplate.h/.cpp` | Eval template + token (Phase 5.4) |
| `Source/LucentEditor/Sequencer/LucentTrackEditor.h/.cpp` | Sequencer UI (Phase 5.5) |

## Lucent-Specific Notes

- `TSoftObjectPtr<UTexture2D>` fields (`DOFKernelTexture`, `DirtMaskTexture`) cannot be
  interpolated — they must snap at Alpha >= 0.5 in `Lerp()`.
- `FLinearColor` fields (bloom layer tints, dirt tint, streaks tint) should interpolate
  per-channel (R/G/B separately), ignoring the Alpha channel.
- `MisregJitter*` range fields (Min/MaxX, Min/MaxY, Min/MaxRot) and `MisregJitterSpeed` are
  legitimate animation targets — jitter bounds can shift over a cut.
- The `Lucent Lens` track is a **master track** (global post-process, no actor binding needed).
  Reference the Tonal track for the same architecture.
- `GlowFormat` (R11G11B10F vs RGBA16F) is a performance-only pipeline switch — do NOT add it
  to `FLucentSettings` or the Sequencer track.

### Cross-DLL CVar Export (applied 2026-02-27)

`LucentCVars.h` and `LucentCVars.cpp` have been updated with `LUCENT_API` on every
`TAutoConsoleVariable<T>` declaration and definition. This is required because `LucentEditor.dll`
must import these globals from `Lucent.dll` across the DLL boundary (Windows
`__declspec(dllexport/dllimport)`). Without it, any file in `LucentEditor` that includes
`LucentCVars.h` produces unresolved external symbol linker errors — one per CVar per object file.

Pattern in `LucentCVars.h`:
```cpp
extern LUCENT_API TAutoConsoleVariable<float> CVarLucentBloomIntensity;
```
Pattern in `LucentCVars.cpp`:
```cpp
LUCENT_API TAutoConsoleVariable<float> CVarLucentBloomIntensity(...);
```

### Sequencer Template: Use ApplyToCVars(), Not Direct CVar Access

The Phase 5.4 template (`FLucentEvalToken::Execute`) calls `Settings.ApplyToCVars()` rather
than accessing CVars directly. **Keep it this way.** The alternative — directly accessing
`CVarLucentXxx->Set(...)` inside the token — would require the Sequencer template file to
include `LucentCVars.h`, creating a cross-DLL dependency from `LucentEditor` to every CVar
symbol. The `ApplyToCVars()` call on the game-thread struct is cleaner and contains the CVar
access inside `Lucent.dll` where it belongs.

For comparison, the Tonal plugin's `MovieSceneTonalSectionTemplate.cpp` accesses CVars directly
(the historical implementation that motivated the TONAL_API/LUCENT_API fix).

## Validation

After all phases are complete, verify:

- [ ] Plugin compiles without error or warning
- [ ] Blueprint: set `Settings.BloomIntensity` on a `ULucentComponent`, call `ApplySettings()`, effect is visible
- [ ] `LerpToPreset()` does not snap any float parameter
- [ ] `ReadFromCVars()` round-trips all fields including the 3 new Phase 1 additions
- [ ] `TSoftObjectPtr` texture fields snap correctly in `Lerp()` (do not attempt to lerp asset refs)
- [ ] Component with `bAutoApply = true`: scrubbing property tracks in the Sequencer updates the viewport without PIE
- [ ] "Add Track" menu in Sequencer shows a "Lucent Lens" entry
- [ ] A Lucent Lens track with two keys shows interpolated lens look when scrubbing between them
- [ ] Dragging a `ULucentPreset` asset onto the timeline creates a key with that preset's settings
- [ ] Movie Render Queue render with a Lucent Lens track produces correct per-frame lens looks
