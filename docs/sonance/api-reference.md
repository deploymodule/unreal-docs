# Sonance — API Reference

## USonanceSubsystem

| Function | Signature | Description |
|---|---|---|
| `PlayMusicTrack` | `void (USonanceMusicTrack*, float BlendDuration)` | Begins playing a music track, blending from the current track. |
| `StopMusic` | `void (float FadeOutDuration)` | Fades out and stops the current music track. |
| `SetStemWeight` | `void (ESonanceStemRole, float Weight, float FadeDuration)` | Adjusts the volume weight of a specific stem role. |
| `PushAmbienceLayer` | `void (USonanceAmbienceLayer*, float BlendIn)` | Activates an ambience layer, blending it in over `BlendIn` seconds. |
| `PopAmbienceLayer` | `void (USonanceAmbienceLayer*, float BlendOut)` | Deactivates an ambience layer, blending it out. |
| `GetMusicIntensity` | `float ()` | Returns the current music intensity value (0.0–1.0). |
| `SetMusicIntensity` | `void (float, float FadeDuration)` | Sets the global music intensity, affecting all stem weights. |

## USonanceDialogueComponent

| Function / Property | Type | Description |
|---|---|---|
| `PlayDialogueData` | `void (USonanceDialogueData*)` | Queues and begins playing all lines in the dialogue data. |
| `StopDialogue` | `void (bool bImmediate)` | Stops current dialogue. If not immediate, waits for the current line. |
| `SkipLine` | `void ()` | Skips to the next line in the current playlist. |
| `IsPlaying` | `bool ()` | Returns true if a dialogue line is currently playing. |
| `OnDialogueComplete` | Delegate | Fired when all lines in the current playlist have played. |
| `OnLineBegin` | Delegate (`FSonanceDialogueLine`) | Fired at the start of each line. |
| `OnLineComplete` | Delegate (`FSonanceDialogueLine`) | Fired at the end of each line. |

## USonanceDialogueData

| Property | Type | Description |
|---|---|---|
| `Lines` | `TArray<FSonanceDialogueLine>` | Ordered list of dialogue lines. |
| `bLoop` | `bool` | If true, the playlist restarts after the last line. |

## FSonanceDialogueLine (Struct)

| Field | Type | Description |
|---|---|---|
| `SubtitleText` | `FText` | Subtitle to display (localized). |
| `AudioPerLocale` | `TMap<FName, USoundBase*>` | Audio asset keyed by locale tag (e.g., `"en"`, `"ja"`). |
| `Priority` | `int32` | Higher priority lines interrupt lower-priority ones when `bInterruptible` is true. |
| `bInterruptible` | `bool` | If false, this line cannot be interrupted by lower or equal priority requests. |
| `MinDelay` | `float` | Minimum delay in seconds before this line plays after the previous one. |

## SonanceFoleyBank (Data Asset)

| Property | Type | Description |
|---|---|---|
| `SurfaceEntries` | `TArray<FSonanceFoleyEntry>` | Per-surface-type foley configuration. |

## FSonanceFoleyEntry (Struct)

| Field | Type | Description |
|---|---|---|
| `SurfaceType` | `TEnumAsByte<EPhysicalSurface>` | Physical surface this entry applies to. |
| `Variations` | `TArray<USoundBase*>` | Pool of audio assets to pick from randomly. |
| `PitchRange` | `FVector2D` | Min/max pitch multiplier. |
| `VolumeRange` | `FVector2D` | Min/max volume multiplier. |

## ESonanceStemRole

| Value | Description |
|---|---|
| `Bass` | Low-frequency stem. |
| `Harmony` | Harmonic/pad stem. |
| `Melody` | Lead melodic stem. |
| `Percussion` | Drum/rhythmic stem. |
| `SFX` | Sound design accent stem. |
