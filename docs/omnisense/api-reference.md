# OmniSense — API Reference

## UOmniSenseComponent

| Property / Function | Type | Description |
|---|---|---|
| `TeamId` | `int32` | Team identifier used to look up relationships in the team matrix. |
| `VisionHalfAngle` | `float` | Half-angle of the vision cone in degrees. |
| `VisionRange` | `float` | Maximum vision detection range in world units. |
| `AcousticSensitivity` | `float` | Multiplier on the acoustic detection radius (1.0 = full radius). |
| `ScentSensitivity` | `float` | Multiplier on scent token detection radius. |
| `bEmitsScent` | `bool` | If true, this component periodically deposits scent tokens. |
| `ScentEmissionInterval` | `float` | Seconds between scent token deposits. |
| `ScentTokenLifetime` | `float` | Seconds before a deposited token decays. |
| `SetScentEmissionEnabled` | `void (bool)` | Enables or disables scent emission at runtime. |
| `GetKnownHostiles` | `TArray<AActor*> ()` | Returns all currently detected hostile actors. |
| `IsActorDetected` | `bool (AActor*)` | Returns true if the given actor is currently in a detected state. |

### Delegates

| Delegate | Parameters | Description |
|---|---|---|
| `OnStimulusDetected` | `FOmniStimulusEvent` | Fired when a new stimulus is first detected. |
| `OnStimulusLost` | `FOmniStimulusEvent` | Fired when a previously detected stimulus leaves detection range. |
| `OnAcousticStimulus` | `FOmniAcousticEvent` | Fired when an acoustic ping is received. |
| `OnScentStimulus` | `FOmniScentEvent` | Fired when a scent token is detected. |

## UOmniSenseSubsystem

| Function | Signature | Description |
|---|---|---|
| `EmitAcousticPing` | `void (FVector Origin, float Radius, float Lifetime, AActor* Source)` | Registers an acoustic ping at the given world location. |
| `RegisterComponent` | `void (UOmniSenseComponent*)` | Registers a component for subsystem management (called automatically on BeginPlay). |
| `UnregisterComponent` | `void (UOmniSenseComponent*)` | Removes a component from the subsystem. |
| `SetTeamMatrix` | `void (UOmniSenseTeamMatrix*)` | Replaces the active team relationship matrix at runtime. |
| `GetRelationship` | `EOmniRelationship (int32 TeamA, int32 TeamB)` | Returns the relationship between two team IDs. |

## FOmniStimulusEvent (Struct)

| Field | Type | Description |
|---|---|---|
| `SourceActor` | `AActor*` | The actor that triggered the stimulus. |
| `SenseType` | `EOmniSenseType` | Which sense detected the stimulus (Vision, Acoustic, Scent). |
| `Relationship` | `EOmniRelationship` | Relationship of the source actor's team to the detecting agent's team. |
| `StimulusLocation` | `FVector` | World location of the stimulus at detection time. |
| `Confidence` | `float` | Detection confidence 0.0–1.0 (affected by occlusion, distance falloff). |

## EOmniSenseType

| Value | Description |
|---|---|
| `Vision` | Line-of-sight frustum check. |
| `Acoustic` | Acoustic ping received. |
| `Scent` | Scent token detected. |

## EOmniRelationship

| Value | Description |
|---|---|
| `Hostile` | Source team is hostile to the detecting team. |
| `Neutral` | No defined relationship. |
| `Friendly` | Source team is allied with the detecting team. |
