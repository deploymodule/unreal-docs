# Sonance — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Sonance** — enable and restart.

## Step 2: Music — Create a Music Track Asset

1. Content Browser > right-click > **Data Asset > SonanceMusicTrack**.
2. Add stem entries: assign a MetaSound source and a `ESonanceStemRole` (Bass, Harmony, Melody, Percussion, SFX) to each.
3. Set `DefaultIntensity` (0.0–1.0) and `Tempo`.

## Step 3: Play Music at Runtime

=== "Blueprint"
    - Call **Sonance > Play Music Track** passing the `USonanceMusicTrack` asset.
    - Call **Sonance > Set Stem Weight** to fade individual stems in/out.

=== "C++"
    ```cpp
    USonanceSubsystem* Sonance = GetWorld()->GetSubsystem<USonanceSubsystem>();
    Sonance->PlayMusicTrack(CombatMusicTrack, 2.0f /* blend duration */);

    // Fade in percussion stem when entering intense combat
    Sonance->SetStemWeight(ESonanceStemRole::Percussion, 1.0f, 1.5f);
    ```

## Step 4: Foley — Set Up a Foley Bank

1. Create a `SonanceFoleyBank` Data Asset.
2. Add entries for each `EPhysicalSurface` value (or use the default UE surface map).
3. Each entry: an array of `USoundBase*` variations, pitch range, volume range.

Assign the bank to your Character Blueprint's **Sonance Foley Bank** property, or set it from C++:

```cpp
USonanceFoleyComponent* Foley = GetComponentByClass<USonanceFoleyComponent>();
Foley->SetFoleyBank(GravelFoleyBank);
```

## Step 5: Dialogue — Create Dialogue Data

1. Create a `USonanceDialogueData` Data Asset.
2. Add `FSonanceDialogueLine` entries:
   - `SubtitleText` (FText)
   - `AudioPerLocale` (TMap of locale FName to USoundBase*)
   - `Priority` (int32)
   - `bInterruptible` (bool)

## Step 6: Play Dialogue

=== "Blueprint"
    1. Add `USonanceDialogueComponent` to your NPC.
    2. Call **Sonance Dialogue > Play Dialogue Data** passing the asset.

=== "C++"
    ```cpp
    #include "Sonance/SonanceDialogueComponent.h"

    DialogueComponent->PlayDialogueData(GreetingDialogueData);

    // Bind to completion
    DialogueComponent->OnDialogueComplete.AddDynamic(this, &AMyNPC::HandleDialogueDone);
    ```

## Step 7: Spatial Ambience

```cpp
USonanceSubsystem* Sonance = GetWorld()->GetSubsystem<USonanceSubsystem>();
// Push an ambience layer (e.g., entering a cave)
Sonance->PushAmbienceLayer(CaveAmbienceLayer, 3.0f /* blend in */);
// Pop when leaving
Sonance->PopAmbienceLayer(CaveAmbienceLayer, 2.0f /* blend out */);
```
