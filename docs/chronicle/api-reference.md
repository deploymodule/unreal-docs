# Chronicle — API Reference

## UChronicleSubsystem

`#include "ChronicleSubsystem.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `StartRecording` | `void StartRecording(AActor* Actor, float Duration, float Rate, bool bSkeletal)` | Begin recording actor transforms |
| `StopRecording` | `void StopRecording(AActor* Actor)` | Stop and clear recording |
| `IsRecording` | `bool IsRecording(AActor* Actor)` | Check if actor is being recorded |
| `BeginRewind` | `void BeginRewind(AActor* Actor, float SpeedMultiplier)` | Start rewinding |
| `StopRewind` | `void StopRewind(AActor* Actor)` | Stop rewind and resume recording |
| `ScrubTo` | `void ScrubTo(AActor* Actor, float NormalizedTime)` | Scrub to position (0–1) |
| `StartReplay` | `void StartReplay(AActor* Actor, float StartOffset, bool bLoop)` | Replay from offset |
| `StopReplay` | `void StopReplay(AActor* Actor)` | Stop replay |
| `GetPlaybackProgress` | `float GetPlaybackProgress(AActor* Actor)` | 0–1 normalized position in buffer |
| `GetBufferedDuration` | `float GetBufferedDuration(AActor* Actor)` | Seconds of recorded history |
| `GetMode` | `EChronicleMode GetMode(AActor* Actor)` | Current mode for actor |

## UChronicleComponent

`#include "ChronicleComponent.h"`

| Property | Type | Description |
|----------|------|-------------|
| `BufferDuration` | float | Seconds of history to retain |
| `RecordingRate` | float | Frames per second to record |
| `bRecordSkeletal` | bool | Record per-bone transforms |
| `bAutoStartRecording` | bool | Begin recording on BeginPlay |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnRewindStarted` | `()` | Rewind begins |
| `OnRewindEnded` | `()` | Rewind stops |
| `OnReplayStarted` | `()` | Replay begins |
| `OnReplayEnded` | `()` | Replay completes |

## EChronicleMode

```cpp
UENUM(BlueprintType)
enum class EChronicleMode : uint8
{
    Idle,
    Recording,
    Rewinding,
    Scrubbing,
    Replaying,
};
```

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Chronicle.DefaultRate` | float | 30.0 | Default recording rate (Hz) |
| `r.Chronicle.DefaultDuration` | float | 10.0 | Default buffer duration (seconds) |
| `r.Chronicle.Debug` | int | 0 | Draw recorded path visualization |
