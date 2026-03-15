# OmniSense — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > OmniSense** — enable and restart.

## Step 2: Add UOmniSenseComponent to Your AI Pawn

=== "Blueprint"
    1. Open your AI Character Blueprint.
    2. **Add Component > OmniSenseComponent**.
    3. Configure **Team**, **Vision Half Angle**, **Vision Range**, and **Acoustic Sensitivity** in Details.

=== "C++"
    ```cpp
    #include "OmniSense/OmniSenseComponent.h"

    UPROPERTY(VisibleAnywhere)
    TObjectPtr<UOmniSenseComponent> OmniSense;

    // Constructor
    OmniSense = CreateDefaultSubobject<UOmniSenseComponent>(TEXT("OmniSense"));
    OmniSense->TeamId = 1;
    OmniSense->VisionHalfAngle = 60.f;
    OmniSense->VisionRange = 1500.f;
    ```

## Step 3: Respond to Sense Events

=== "Blueprint"
    Bind **On Stimulus Detected** on the OmniSense component in your AI Character's BeginPlay:
    ```
    [BeginPlay] → [Bind OnStimulusDetected] → [AI Event: React to Stimulus]
    ```

=== "C++"
    ```cpp
    OmniSense->OnStimulusDetected.AddDynamic(this, &AMyAICharacter::HandleStimulusDetected);

    void AMyAICharacter::HandleStimulusDetected(const FOmniStimulusEvent& Event)
    {
        if (Event.SenseType == EOmniSenseType::Vision && Event.Relationship == EOmniRelationship::Hostile)
        {
            AIController->SetFocus(Event.SourceActor);
        }
    }
    ```

## Step 4: Emit Acoustic Pings

```cpp
// Emit a gunshot sound ping
UOmniSenseSubsystem* Subsystem = GetWorld()->GetSubsystem<UOmniSenseSubsystem>();
Subsystem->EmitAcousticPing(GetActorLocation(), 2000.f, 1.5f, this);
```

## Step 5: Emit Scent Tokens

```cpp
// Start passive scent emission (e.g., on a patrolling enemy)
OmniSense->SetScentEmissionEnabled(true);
OmniSense->ScentEmissionInterval = 0.5f;
OmniSense->ScentTokenLifetime = 30.f;
```

## Step 6: Configure the Team Matrix

1. Content Browser > right-click > **Data Asset > OmniSenseTeamMatrix**.
2. Add rows for each team pair and set relationship (Hostile/Neutral/Friendly).
3. Assign the asset to **OmniSenseSubsystem > Team Matrix** in Project Settings.
