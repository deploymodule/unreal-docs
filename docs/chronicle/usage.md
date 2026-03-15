# Chronicle — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Chronicle"` to your `Build.cs`.

## 2. Start Recording an Actor

```cpp
UChronicleSubsystem* Chronicle = GetWorld()->GetSubsystem<UChronicleSubsystem>();

// Start recording MyActor with a 10-second buffer at 30 Hz
Chronicle->StartRecording(MyActor, 10.f, 30.f);
```

Or add `UChronicleComponent` to an actor — it will auto-register on `BeginPlay`:

```cpp
ChronicleComponent = CreateDefaultSubobject<UChronicleComponent>(TEXT("Chronicle"));
ChronicleComponent->BufferDuration = 10.f;
ChronicleComponent->RecordingRate = 30.f;
```

## 3. Trigger Rewind

```cpp
// Rewind at 2x speed (backward)
Chronicle->BeginRewind(MyActor, 2.0f);

// Stop rewind and resume recording
Chronicle->StopRewind(MyActor);
```

## 4. Scrub to a Specific Time

```cpp
// 0.0 = oldest recorded frame, 1.0 = current moment
Chronicle->ScrubTo(MyActor, 0.5f);   // jump to the midpoint
```

## 5. Replay from a Time Offset

```cpp
// Replay the last 5 seconds
Chronicle->StartReplay(MyActor, -5.f, false);  // bLoop = false

// Loop replay (ghost mode)
Chronicle->StartReplay(MyActor, -10.f, true);
```

## 6. Record Skeletal Pose

```cpp
// Enable full skeletal recording (heavier)
Chronicle->StartRecording(MyCharacter, 8.f, 30.f, true /* bRecordSkeletal */);
```

## 7. Get Playback Progress

```cpp
float Progress = Chronicle->GetPlaybackProgress(MyActor);  // 0–1
float BufferedDuration = Chronicle->GetBufferedDuration(MyActor);
```

## 8. Bind to Events

```cpp
ChronicleComponent->OnRewindStarted.AddDynamic(this, &AMyChar::OnRewindStarted);
ChronicleComponent->OnRewindEnded.AddDynamic(this, &AMyChar::OnRewindEnded);
```

## 9. Stop Recording

```cpp
Chronicle->StopRecording(MyActor);
```
