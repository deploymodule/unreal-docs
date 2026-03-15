# Tonal: Animation, Sequencer & Runtime Control — Implementation Plan

## Context

Tonal is an Unreal Engine 5.7 post-process rendering plugin (compute shader pipeline via
`FSceneViewExtensionBase`). Parameters live as CVars at render time; `FTonalSettings` is a
`USTRUCT(BlueprintType)` that mirrors CVars for game-thread access. `UTonalComponent` is a
`BlueprintSpawnableComponent` that owns `FTonalSettings` and pushes it to CVars via
`bAutoApply` tick or explicit `ApplySettings()`.

## Goal

Full support for:
- **Sequencer animation** — keyframe any parameter in the editor timeline, with working viewport preview
- **Blueprint control** — every parameter reachable via BP nodes
- **C++ runtime control** — structured API on top of the CVar layer
- **Smooth transitions** — lerp between named presets or arbitrary snapshots at runtime

---

## Todo List

### Phase 1 — Complete `FTonalSettings` (closes the CVar gap)

**Status: Not started**

`FTonalSettings` covers ~60% of `FFrameData`. The following feature groups exist in CVars /
`FFrameData` but are absent from the settings struct, making them unreachable from Blueprint
and Sequencer. Add each group to:
1. `FTonalSettings` as `UPROPERTY(EditAnywhere, BlueprintReadWrite, ...)` fields
2. `FTonalSettings::ApplyToCVars()` — push to CVars
3. `FTonalSettings::ReadFromCVars()` — pull from CVars
4. `FTonalSettings::Lerp()` — interpolate (floats linear, bools snap at 0.5, enums snap at 0.5)

#### 1.1 Film Vignette

Fields to add to `FTonalSettings` (matching `FFrameData` and the `r.Tonal.FilmVignette.*` CVars):

```cpp
// -- Film Vignette --
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette")
bool bFilmVignetteEnabled = false;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette",
    meta = (ClampMin="0", ClampMax="1", EditCondition="bFilmVignetteEnabled",
            ToolTip="0=Spherical, 1=Anamorphic (horizontal squeeze)."))
int32 FilmVignetteMode = 0;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bFilmVignetteEnabled"))
float FilmVignetteStrength = 0.5f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bFilmVignetteEnabled"))
float FilmVignetteInnerRadius = 0.3f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette",
    meta = (ClampMin="0.0", ClampMax="2.0", EditCondition="bFilmVignetteEnabled"))
float FilmVignetteOuterRadius = 1.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette",
    meta = (ClampMin="-1.0", ClampMax="1.0", EditCondition="bFilmVignetteEnabled"))
float FilmVignetteCenterX = 0.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette",
    meta = (ClampMin="-1.0", ClampMax="1.0", EditCondition="bFilmVignetteEnabled"))
float FilmVignetteCenterY = 0.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette",
    meta = (ClampMin="0.1", ClampMax="4.0", EditCondition="bFilmVignetteEnabled"))
float FilmVignetteAspectRatio = 1.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette",
    meta = (ClampMin="0.5", ClampMax="8.0", EditCondition="bFilmVignetteEnabled"))
float FilmVignetteFalloffCurve = 2.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Vignette",
    meta = (EditCondition="bFilmVignetteEnabled",
            ToolTip="Color tint blended at vignette edges. Default is a cool blue-grey."))
FLinearColor FilmVignetteTint = FLinearColor(0.90f, 0.92f, 1.00f, 1.0f);
```

`ApplyToCVars()` additions:
```cpp
CVarTonalFilmVignetteEnabled->Set(bFilmVignetteEnabled ? 1 : 0, ECVF_SetByConsole);
CVarTonalFilmVignetteMode->Set(FilmVignetteMode, ECVF_SetByConsole);
CVarTonalFilmVignetteStrength->Set(FilmVignetteStrength, ECVF_SetByConsole);
CVarTonalFilmVignetteInnerRadius->Set(FilmVignetteInnerRadius, ECVF_SetByConsole);
CVarTonalFilmVignetteOuterRadius->Set(FilmVignetteOuterRadius, ECVF_SetByConsole);
CVarTonalFilmVignetteCenterX->Set(FilmVignetteCenterX, ECVF_SetByConsole);
CVarTonalFilmVignetteCenterY->Set(FilmVignetteCenterY, ECVF_SetByConsole);
CVarTonalFilmVignetteAspectRatio->Set(FilmVignetteAspectRatio, ECVF_SetByConsole);
CVarTonalFilmVignetteFalloffCurve->Set(FilmVignetteFalloffCurve, ECVF_SetByConsole);
CVarTonalFilmVignetteTintR->Set(FilmVignetteTint.R, ECVF_SetByConsole);
CVarTonalFilmVignetteTintG->Set(FilmVignetteTint.G, ECVF_SetByConsole);
CVarTonalFilmVignetteTintB->Set(FilmVignetteTint.B, ECVF_SetByConsole);
```

`ReadFromCVars()` and `Lerp()`: follow same pattern as existing sections.

---

#### 1.2 Film White Balance

Fields to add:

```cpp
// -- Film White Balance --
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film White Balance",
    meta = (ClampMin="0", ClampMax="4",
            ToolTip="0=Custom (uses WBTemp), 1=Tungsten 3200K, 2=Fluorescent 4000K, 3=Daylight 5600K, 4=Overcast 7500K."))
int32 FilmWBPreset = 0;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film White Balance",
    meta = (ClampMin="2000.0", ClampMax="12000.0",
            EditCondition="FilmWBPreset == 0",
            ToolTip="Color temperature in Kelvin. Used when WBPreset is Custom."))
float FilmWBTemp = 6500.0f;
```

`ApplyToCVars()`:
```cpp
CVarTonalFilmWBPreset->Set(FilmWBPreset, ECVF_SetByConsole);
CVarTonalFilmWBTemp->Set(FilmWBTemp, ECVF_SetByConsole);
```

`Lerp()`: `FilmWBPreset` snaps at 0.5; `FilmWBTemp` lerps linearly.

---

#### 1.3 Film Stock — Tonemapper Respect & Push/Pull

Fields to add:

```cpp
// -- Film Stock Modifiers --
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Stock (HDR)",
    meta = (EditCondition="bFilmStockEnabled",
            ToolTip="Scale Film Stock shoulder based on UE tonemapper's ACES shoulder setting."))
bool bFilmRespectTonemapper = false;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Film Stock (HDR)",
    meta = (EditCondition="bFilmStockEnabled",
            ToolTip="Push/pull shoulder and toe based on Auto Exposure bias."))
bool bFilmPushPullEnabled = false;
```

`ApplyToCVars()`:
```cpp
CVarTonalFilmStockRespectTonemapper->Set(bFilmRespectTonemapper ? 1 : 0, ECVF_SetByConsole);
CVarTonalFilmStockPushPull->Set(bFilmPushPullEnabled ? 1 : 0, ECVF_SetByConsole);
```

---

#### 1.4 VHS Tape Emulation

Fields to add (matching `r.Tonal.VHS.*` CVars):

```cpp
// -- VHS Tape Emulation --
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS")
bool bVHSEnabled = false;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bVHSEnabled"))
float VHSBlend = 1.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="16.0", EditCondition="bVHSEnabled",
            ToolTip="Horizontal chroma channel offset in pixels."))
float VHSChromaShift = 4.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="8.0", EditCondition="bVHSEnabled"))
float VHSChromaBlur = 2.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bVHSEnabled"))
float VHSLumaNoise = 0.05f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bVHSEnabled"))
float VHSChromaNoise = 0.04f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bVHSEnabled",
            ToolTip="Scanline darkening intensity."))
float VHSScanlines = 0.15f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.5", ClampMax="4.0", EditCondition="bVHSEnabled",
            ToolTip="Scanline frequency multiplier. 1.0 = one line per pixel."))
float VHSScanlineFreq = 1.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bVHSEnabled",
            ToolTip="Horizontal band distortion strength at the bottom of the frame."))
float VHSHeadSwitch = 0.3f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="0.3", EditCondition="bVHSEnabled",
            ToolTip="Fraction of frame height occupied by the head-switch zone."))
float VHSHeadSwitchZone = 0.05f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bVHSEnabled",
            ToolTip="Additive colour cast (greenish tinge) blend amount."))
float VHSColorCast = 0.3f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|VHS",
    meta = (ClampMin="0.0", ClampMax="0.5", EditCondition="bVHSEnabled",
            ToolTip="Luma quantization crush. Bands the luminance into discrete steps."))
float VHSLumaCrush = 0.08f;
```

`ApplyToCVars()`:
```cpp
CVarTonalVHSEnabled->Set(bVHSEnabled ? 1 : 0, ECVF_SetByConsole);
CVarTonalVHSBlend->Set(VHSBlend, ECVF_SetByConsole);
CVarTonalVHSChromaShift->Set(VHSChromaShift, ECVF_SetByConsole);
CVarTonalVHSChromaBlur->Set(VHSChromaBlur, ECVF_SetByConsole);
CVarTonalVHSLumaNoise->Set(VHSLumaNoise, ECVF_SetByConsole);
CVarTonalVHSChromaNoise->Set(VHSChromaNoise, ECVF_SetByConsole);
CVarTonalVHSScanlines->Set(VHSScanlines, ECVF_SetByConsole);
CVarTonalVHSScanlineFreq->Set(VHSScanlineFreq, ECVF_SetByConsole);
CVarTonalVHSHeadSwitch->Set(VHSHeadSwitch, ECVF_SetByConsole);
CVarTonalVHSHeadSwitchZone->Set(VHSHeadSwitchZone, ECVF_SetByConsole);
CVarTonalVHSColorCast->Set(VHSColorCast, ECVF_SetByConsole);
CVarTonalVHSLumaCrush->Set(VHSLumaCrush, ECVF_SetByConsole);
```

`Lerp()`: all float fields lerp; `bVHSEnabled` snaps at 0.5.

Note: `VHSTime` is NOT included here — it is packed from real-time seconds in `BeginRenderViewFamily`, not a user-settable parameter.

---

#### 1.5 Missing LightWrap Parameters

The following exist in `FFrameData` and CVars but are missing from `FTonalSettings`:

```cpp
// In Tonal|Light Wrap section (alongside existing fields):
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Light Wrap",
    meta = (ClampMin="0.0", ClampMax="1000.0", EditCondition="bLightWrapEnabled",
            ToolTip="Minimum depth discontinuity (cm) to be counted as an edge."))
float LightWrapMinDepthDiff = 50.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Light Wrap",
    meta = (ClampMin="0.0", ClampMax="10000.0", EditCondition="bLightWrapEnabled",
            ToolTip="Scene-space max distance for wrap contribution. 0 = no limit."))
float LightWrapMaxDistance = 0.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Light Wrap",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bLightWrapEnabled",
            ToolTip="Edge mask feather amount. Softens the boundary of the wrap region."))
float LightWrapFeather = 0.0f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Light Wrap",
    meta = (ClampMin="0.0", ClampMax="16.0", EditCondition="bLightWrapEnabled",
            ToolTip="Pixel dilation of the edge mask before blurring."))
float LightWrapExpand = 4.0f;
```

`ApplyToCVars()`:
```cpp
CVarTonalLightWrapMinDepthDiff->Set(LightWrapMinDepthDiff, ECVF_SetByConsole);
CVarTonalLightWrapMaxDistance->Set(LightWrapMaxDistance, ECVF_SetByConsole);
CVarTonalLightWrapFeather->Set(LightWrapFeather, ECVF_SetByConsole);
CVarTonalLightWrapExpand->Set(LightWrapExpand, ECVF_SetByConsole);
```

---

#### 1.6 Missing LocalTone Parameters

```cpp
// In Tonal|Local Tone Mapping section:
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Local Tone Mapping",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bLocalToneEnabled",
            ToolTip="How much to compress highlight detail layer back toward base."))
float LocalToneCompressionAmount = 0.3f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Local Tone Mapping",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bLocalToneEnabled"))
float LocalToneHighlightThreshold = 0.7f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tonal|Local Tone Mapping",
    meta = (ClampMin="0.0", ClampMax="1.0", EditCondition="bLocalToneEnabled",
            ToolTip="Global blend between unprocessed input and local tone result."))
float LocalToneBlend = 1.0f;
```

`ApplyToCVars()`:
```cpp
CVarTonalLocalToneCompressionAmount->Set(LocalToneCompressionAmount, ECVF_SetByConsole);
CVarTonalLocalToneHighlightThreshold->Set(LocalToneHighlightThreshold, ECVF_SetByConsole);
CVarTonalLocalToneBlend->Set(LocalToneBlend, ECVF_SetByConsole);
```

---

### Phase 2 — Enable Editor-Tick for Sequencer Preview

**Status: Not started**

#### 2.1 Mark `UTonalComponent` to tick in editor

In `TonalComponent.h`, change the class specifier:

```cpp
UCLASS(Blueprintable, ClassGroup=(Rendering),
    meta=(BlueprintSpawnableComponent, bTickInEditor=true),
    HideCategories=(Collision, Physics, Navigation))
class TONAL_API UTonalComponent : public UActorComponent
```

In the constructor in `TonalComponent.cpp`:

```cpp
PrimaryComponentTick.bCanEverTick = true;
PrimaryComponentTick.bStartWithTickEnabled = false;
// Add:
PrimaryComponentTick.bTickEvenWhenPaused = false;
```

Also override `TickComponent` to guard against editor-mode ticking when `bAutoApply` is off
(already gated by the `if (bAutoApply)` check — verify this is sufficient).

#### 2.2 Sequencer scrubbing: ensure tick fires during evaluation

When `bAutoApply` is true and `bTickInEditor` is set, Sequencer scrubbing in the editor will
cause the component to tick, which calls `ApplySettings()` → `ApplyToCVars()` → GPU sees
updated values. Verify this works by testing Sequencer preview in a non-PIE viewport.

---

### Phase 3 — Expand `UTonalComponent` Blueprint API

**Status: Not started**

#### 3.1 Add convenience setters for all new Phase 1 parameters

Follow the same pattern as existing `SetShaperContrast` / `SetMicroContrastStrength`:

```cpp
// Film Vignette
UFUNCTION(BlueprintCallable, Category = "Tonal|Film Vignette")
void SetFilmVignetteStrength(float Value);

UFUNCTION(BlueprintCallable, Category = "Tonal|Film Vignette")
void SetFilmVignetteTint(FLinearColor Tint);

// White Balance
UFUNCTION(BlueprintCallable, Category = "Tonal|Film White Balance")
void SetFilmWBPreset(int32 Preset);

UFUNCTION(BlueprintCallable, Category = "Tonal|Film White Balance")
void SetFilmWBTemp(float Kelvin);

// VHS
UFUNCTION(BlueprintCallable, Category = "Tonal|VHS")
void SetVHSEnabled(bool bEnabled);

UFUNCTION(BlueprintCallable, Category = "Tonal|VHS")
void SetVHSBlend(float Value);

// ... (one per most-used VHS param at minimum)
```

Each setter should update both the `Settings` field and the CVar directly (same pattern as
existing setters).

#### 3.2 Add a `SnapshotToCurrent()` helper

```cpp
/** Capture current CVar state into this component's Settings without applying. Useful for
    setting up a Lerp baseline from whatever is currently live. */
UFUNCTION(BlueprintCallable, Category = "Tonal")
void SnapshotFromCurrent();
```

Implementation: calls `Settings.ReadFromCVars()`.

#### 3.3 Extend `LerpToSettings` / `LerpToPreset` to use a curve

Optional quality upgrade:

```cpp
UFUNCTION(BlueprintCallable, Category = "Tonal")
void LerpToSettingsCurved(const FTonalSettings& Target, float Alpha, UCurveFloat* Curve);
```

Evaluates `Curve` at `Alpha` to remap the interpolation shape (ease-in, ease-out, etc.) before
passing to the existing `Lerp()`.

---

### Phase 4 — Verify `Lerp()` completeness

**Status: Not started**

After Phase 1 adds new fields to `FTonalSettings`, audit `FTonalSettings::Lerp()` in
`TonalSettings.cpp` to confirm every new field is interpolated. Missing fields in `Lerp()`
will cause snapping rather than smooth animation when using `LerpToSettings()` /
`LerpToPreset()`.

Checklist per new section:
- [ ] Film Vignette — all floats lerp, bool/int snap at 0.5
- [ ] Film WB — `FilmWBTemp` lerps, `FilmWBPreset` snaps
- [ ] Film modifiers — bools snap
- [ ] VHS — all floats lerp, bool snaps
- [ ] LightWrap additions — all lerp
- [ ] LocalTone additions — all lerp

---

### Phase 5 — Custom Sequencer Track

**Status: Not started**

This is the primary Sequencer integration. It gives artists a dedicated "Tonal Look" track in
the Sequencer timeline, frame-accurate evaluation (no tick dependency), and in-editor viewport
preview while scrubbing. It does not require `bAutoApply` to be enabled on any component.

#### Design: Snapshot-Key Model

The track uses a **snapshot-per-key** data model rather than per-channel float curves.
Each key stores a complete `FTonalSettings` value. At evaluation time the two bracketing keys
are found and `FTonalSettings::Lerp()` interpolates between them. This is the right model for
a look tool — artists think in whole presets ("Look A → Look B"), not individual float curves.

---

#### 5.1 New key struct — `Source/TonalEditor/Sequencer/TonalSettingsKey.h`

```cpp
#pragma once
#include "TonalSettings.h"
#include "TonalSettingsKey.generated.h"

/** A single timed look keyframe for the Tonal Sequencer track. */
USTRUCT()
struct FTonalSettingsKey
{
    GENERATED_BODY()

    UPROPERTY()
    FFrameNumber Time;

    UPROPERTY()
    FTonalSettings Value;
};
```

---

#### 5.2 Section — `Source/TonalEditor/Sequencer/MovieSceneTonalSection.h/.cpp`

```cpp
UCLASS()
class UTonalSection : public UMovieSceneSection
{
    GENERATED_BODY()
public:
    UTonalSection();

    /** Keys sorted ascending by Time. */
    UPROPERTY()
    TArray<FTonalSettingsKey> Keys;

    /** Add or overwrite a key at the given frame. */
    void AddKey(FFrameNumber Time, const FTonalSettings& InValue);

    /** Evaluate the interpolated look at the given time.
        Returns default settings if the section has no keys. */
    FTonalSettings Evaluate(FFrameTime Time) const;
};
```

`Evaluate()` logic:
- Empty → return `FTonalSettings{}`
- One key, or time ≤ first key → return `Keys[0].Value`
- Time ≥ last key → return `Keys.Last().Value`
- Otherwise: find bracketing pair `[i]` / `[i+1]`, compute
  `Alpha = (Time.AsDecimal() - Keys[i].Time.Value) / float(Keys[i+1].Time.Value - Keys[i].Time.Value)`,
  return `FTonalSettings::Lerp(Keys[i].Value, Keys[i+1].Value, Alpha)`

---

#### 5.3 Track — `Source/TonalEditor/Sequencer/MovieSceneTonalTrack.h/.cpp`

```cpp
UCLASS(MinimalAPI, BlueprintType)
class UMovieSceneTonalTrack : public UMovieSceneNameableTrack
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
    { return NSLOCTEXT("Tonal", "TrackName", "Tonal Look"); }
#endif

private:
    UPROPERTY()
    TArray<TObjectPtr<UMovieSceneSection>> Sections;
};
```

`CreateNewSection()` allocates a `UTonalSection` and sets its range to the owning sequence's
full playback range.

`CreateTemplateForSection()` returns a `FTonalSectionTemplate` (5.4).

---

#### 5.4 Evaluation template — `Source/TonalEditor/Sequencer/MovieSceneTonalSectionTemplate.h/.cpp`

```cpp
// Execution token — runs on game thread, applies evaluated settings to CVars.
struct FTonalEvalToken : IMovieSceneExecutionToken
{
    FTonalSettings Settings;

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
struct FTonalSectionTemplate : FMovieSceneEvalTemplate
{
    GENERATED_BODY()

    FTonalSectionTemplate() = default;
    explicit FTonalSectionTemplate(const UTonalSection& Section);

    UPROPERTY()
    TWeakObjectPtr<const UTonalSection> SectionPtr;

    virtual UScriptStruct& GetScriptStructImpl() const override { return *StaticStruct(); }

    virtual void Evaluate(const FMovieSceneEvaluationOperand&,
                          const FMovieSceneContext& Context,
                          const FPersistentEvaluationData&,
                          FMovieSceneExecutionTokens& ExecutionTokens) const override
    {
        if (const UTonalSection* Section = SectionPtr.Get())
        {
            FTonalEvalToken Token;
            Token.Settings = Section->Evaluate(Context.GetTime());
            ExecutionTokens.Add(MoveTemp(Token));
        }
    }
};
```

The correct base header in UE5.7 is `MovieSceneEvalTemplate.h` under
`MovieScene/Public/Evaluation/`. Confirm against `FMovieSceneCameraShakeSectionTemplate` as
a real-engine reference for the same pattern.

---

#### 5.5 Track editor — `Source/TonalEditor/Sequencer/TonalTrackEditor.h/.cpp`

```cpp
class FTonalTrackEditor : public FMovieSceneTrackEditor
{
public:
    static TSharedRef<ISequencerTrackEditor> CreateTrackEditor(TSharedRef<ISequencer> Sequencer);
    explicit FTonalTrackEditor(TSharedRef<ISequencer> InSequencer);

    virtual bool SupportsType(TSubclassOf<UMovieSceneTrack> TrackClass) const override;
    virtual void BuildAddTrackMenu(FMenuBuilder& MenuBuilder) override;
    virtual bool HandleAssetAdded(UObject* Asset, const FGuid& TargetObjectGuid) override;

private:
    void AddTonalTrack();
    void AddKeyAtCurrentTime(UTonalSection* Section);
};
```

Behaviours:
- `BuildAddTrackMenu`: adds "Tonal Look" under the master tracks section; calls `AddTonalTrack()`.
  The track is a **global master track** (add via `UMovieScene::AddMasterTrack<UMovieSceneTonalTrack>()`).
  No object binding needed — Tonal is a global post-process effect.
- `HandleAssetAdded`: if a `UTonalPreset` is dragged onto the timeline, create a key at the
  current time with `Preset->Settings` as the value.
- `AddKeyAtCurrentTime`: calls `FTonalSettings::ReadFromCVars()` to snapshot the current live
  state, then calls `UTonalSection::AddKey()`.

---

#### 5.6 Module dependencies — `TonalEditor.Build.cs`

Add to `PrivateDependencyModuleNames`:

```csharp
"MovieScene",
"MovieSceneTracks",
"Sequencer",
"SequencerCore",
```

---

#### 5.7 Registration in `FTonalEditorModule`

In `TonalEditor.h`, add private member:

```cpp
FDelegateHandle TrackEditorHandle;
```

In `FTonalEditorModule::StartupModule()`, after the existing tab/preset setup:

```cpp
ISequencerModule& SequencerModule =
    FModuleManager::LoadModuleChecked<ISequencerModule>("Sequencer");
TrackEditorHandle = SequencerModule.RegisterTrackEditor(
    FOnCreateTrackEditor::CreateStatic(&FTonalTrackEditor::CreateTrackEditor));
```

In `FTonalEditorModule::ShutdownModule()`:

```cpp
if (ISequencerModule* SequencerModule =
    FModuleManager::GetModulePtr<ISequencerModule>("Sequencer"))
{
    SequencerModule->UnRegisterTrackEditor(TrackEditorHandle);
}
```

---

## Implementation Order

1. **Phase 1** — Complete `FTonalSettings`. Do all six subsections (1.1–1.6) in one pass
   through `TonalSettings.h` and `TonalSettings.cpp`. This is the prerequisite for everything.

2. **Phase 4** — Immediately after Phase 1, audit `Lerp()`. Do this while the new fields are
   fresh.

3. **Phase 2** — One-line `bTickInEditor` change. Do alongside Phase 1.

4. **Phase 3** — Convenience setters. Do after Phase 1 compiles cleanly.

5. **Phase 5** — Build the Sequencer track. Phases 1 and 4 must be complete first so
   `FTonalSettings::Lerp()` is correct before the section's `Evaluate()` calls it.
   Work in order: 5.1 → 5.2 → 5.3 → 5.4 → 5.5 → 5.6 → 5.7.

---

## Key Files

| File | Role |
|---|---|
| `Source/Tonal/TonalSettings.h` | Add all new UPROPERTY fields (Phases 1, 4) |
| `Source/Tonal/TonalSettings.cpp` | `ApplyToCVars`, `ReadFromCVars`, `Lerp` additions (Phases 1, 4) |
| `Source/Tonal/TonalCVars.h` | Verify all referenced CVars are declared (should already exist) |
| `Source/Tonal/TonalComponent.h` | `bTickInEditor=true`, new UFUNCTION setters (Phases 2, 3) |
| `Source/Tonal/TonalComponent.cpp` | Implement new setters (Phase 3) |
| `Source/TonalEditor/TonalEditor.h` | Add `TrackEditorHandle` field (Phase 5.7) |
| `Source/TonalEditor/TonalEditor.cpp` | Register/unregister track editor (Phase 5.7) |
| `Source/TonalEditor/TonalEditor.Build.cs` | Add MovieScene/Sequencer deps (Phase 5.6) |
| `Source/TonalEditor/Sequencer/TonalSettingsKey.h` | Key struct (Phase 5.1) |
| `Source/TonalEditor/Sequencer/MovieSceneTonalSection.h/.cpp` | Section + Evaluate() (Phase 5.2) |
| `Source/TonalEditor/Sequencer/MovieSceneTonalTrack.h/.cpp` | Track asset (Phase 5.3) |
| `Source/TonalEditor/Sequencer/MovieSceneTonalSectionTemplate.h/.cpp` | Eval template + token (Phase 5.4) |
| `Source/TonalEditor/Sequencer/TonalTrackEditor.h/.cpp` | Sequencer UI (Phase 5.5) |

## Validation

After all phases are complete, verify:

- [ ] Plugin compiles without error or warning
- [ ] Blueprint: set `Settings.VHSBlend` on a `UTonalComponent`, call `ApplySettings()`, effect is visible
- [ ] `LerpToPreset()` does not snap any float parameter including new VHS/Vignette fields
- [ ] `ReadFromCVars()` round-trips all new fields (set via console, read back matches)
- [ ] Component with `bAutoApply = true`: scrubbing property tracks in the Sequencer updates the viewport without PIE
- [ ] "Add Track" menu in Sequencer shows a "Tonal Look" entry
- [ ] A Tonal Look track with two keys shows interpolated look when scrubbing between them
- [ ] Dragging a `UTonalPreset` asset onto the timeline creates a key with that preset's settings
- [ ] Movie Render Queue render with a Tonal Look track produces correct per-frame looks
