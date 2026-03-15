# Chronicle — Overview

## Architecture

```
UChronicleSubsystem  (UWorldSubsystem)
    └── TMap<AActor*, FChronicleTrack>   Tracks
            └── TCircularBuffer<FChronicleFrame>  FrameBuffer

FChronicleFrame  (struct)
    ├── float          Timestamp
    ├── FVector        Location
    ├── FQuat          Rotation
    └── FVector        Scale

UChronicleComponent  (UActorComponent)
    ├── bool           bRecording
    ├── EChronicleMode Mode    (Recording / Rewinding / Scrubbing / Replaying)
    └── float          PlaybackTime
```

## Recording

The subsystem ticks every frame (or at a configurable fixed rate) and pushes a new `FChronicleFrame` into the actor's ring buffer. The ring buffer capacity is derived from `BufferDuration × RecordingRate`.

Default settings:
- **Recording rate**: 30 Hz
- **Buffer duration**: 10 seconds
- **Total frames per actor**: 300

Frames older than `BufferDuration` are overwritten.

## Playback Modes

### Rewind

The actor's transform is driven backward through the frame buffer at a configurable speed multiplier. Physics and collision can optionally be disabled during rewind.

### Scrub

The caller controls `PlaybackTime` directly (0 = oldest, 1 = newest). Useful for implementing slow-motion or a rewind bar UI.

### Replay

The actor plays forward from a specified time offset, either once or looping. Useful for ghost replays, training dummies, or replaying a player's run.

## Interpolation

Between frames, Chronicle uses cubic Hermite interpolation for smooth position and slerp for rotation. This ensures rewound motion is smooth even when the recording rate is lower than the display frame rate.

## Skeletal Mesh Recording

Chronicle can optionally record per-bone transforms for skeletal meshes via `FChronicleSkeletalTrack`. This is more expensive but enables full body rewind (pose included).

## Multiple Actor Management

All registered actors share one subsystem tick. The tick iterates the track map and records each actor's transform. This avoids N separate component ticks.
