# Perpetual — Usage

## Step 1: Create a PerpetualAnimationAsset

1. Content Browser > right-click > **Miscellaneous > Data Asset > PerpetualAnimationAsset**.
2. Add stages to the **Stages** array. Each stage has:
   - **Stage Name** (e.g., `Opening`, `Open`, `Closing`)
   - **Target Transform** (relative to actor root)
   - **Duration** (seconds)
   - **Ease Curve** (optional `UCurveFloat`)
   - **Next Stage** (optional auto-chain)
   - **Audio Cue** / **Particle System** to fire at stage start/end

## Step 2: Add UPerpetualAnimationComponent to Your Actor

=== "Blueprint"
    1. Open your door/lever/switch Blueprint.
    2. **Add Component > PerpetualAnimationComponent**.
    3. Set **Animation Asset** in Details to your Data Asset.
    4. Set **Target Component** to the mesh or scene component to animate.

=== "C++"
    ```cpp
    #include "Perpetual/PerpetualAnimationComponent.h"

    UPROPERTY(VisibleAnywhere)
    TObjectPtr<UPerpetualAnimationComponent> PerpetualAnim;

    // In constructor
    PerpetualAnim = CreateDefaultSubobject<UPerpetualAnimationComponent>(TEXT("PerpetualAnim"));
    ```

## Step 3: Trigger Stage Transitions

=== "Blueprint"
    - Call **Perpetual > Go To Stage** and pass the stage name (e.g., `"Opening"`).
    - Call **Perpetual > Advance** to move to the next stage in the sequence.
    - Call **Perpetual > Reverse** to go to the configured reverse stage.

=== "C++"
    ```cpp
    // Trigger door open
    PerpetualAnim->GoToStage(FName("Opening"));

    // Advance through sequence
    PerpetualAnim->Advance();

    // Reverse (e.g., close door)
    PerpetualAnim->Reverse();
    ```

## Step 4: Bind to Events

=== "Blueprint"
    Bind to the `OnStageComplete` delegate in the Event Graph:
    ```
    [BeginPlay] → [Bind Event to OnStageComplete] → [Custom Event: HandleStageComplete]
    ```

=== "C++"
    ```cpp
    PerpetualAnim->OnStageComplete.AddDynamic(this, &AMyDoor::HandleStageComplete);

    void AMyDoor::HandleStageComplete(FName StageName)
    {
        if (StageName == FName("Open"))
        {
            // Door is fully open
        }
    }
    ```

## Example: Auto-Closing Door

Set the **Open** stage's **Next Stage** to `"Closing"` and **Next Stage Delay** to `3.0`. The door will automatically begin closing 3 seconds after it finishes opening — no Blueprint required.

## Example: Looping Fan

Create a single stage with **Loop** enabled, **Duration** = `2.0`, and **Target Rotation** = `(0, 0, 360)`. The fan spins indefinitely.
