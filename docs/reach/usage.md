# Reach — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Reach** — enable and restart.

## Step 2: Add UReachDetectorComponent to Your Player

=== "Blueprint"
    1. Open your Player Character Blueprint.
    2. **Add Component > ReachDetectorComponent**.
    3. Set **Detection Radius**, **Detection Interval**, and **Interact Input Action**.

=== "C++"
    ```cpp
    #include "Reach/ReachDetectorComponent.h"

    UPROPERTY(VisibleAnywhere)
    TObjectPtr<UReachDetectorComponent> ReachDetector;

    // Constructor
    ReachDetector = CreateDefaultSubobject<UReachDetectorComponent>(TEXT("ReachDetector"));
    ReachDetector->DetectionRadius = 200.f;
    ReachDetector->DetectionInterval = 0.1f;
    ```

## Step 3: Add UReachInteractableComponent to a Prop

=== "Blueprint"
    1. Open your prop Blueprint (door, chest, button, etc.).
    2. **Add Component > ReachInteractableComponent**.
    3. Set **Interaction Mode** (Instant, Hold, Smash, Rhythm) and **Display Name**.

=== "C++"
    ```cpp
    #include "Reach/ReachInteractableComponent.h"

    UPROPERTY(VisibleAnywhere)
    TObjectPtr<UReachInteractableComponent> Interactable;

    Interactable = CreateDefaultSubobject<UReachInteractableComponent>(TEXT("Interactable"));
    Interactable->InteractionMode = EReachInteractionMode::Hold;
    Interactable->HoldDuration = 2.0f;
    ```

## Step 4: Respond to Interaction Events

=== "Blueprint"
    Bind **On Interacted** on the `UReachInteractableComponent`:
    ```
    [BeginPlay] → [Bind OnInteracted] → [Open Door / Pick Up Item / etc.]
    ```

=== "C++"
    ```cpp
    Interactable->OnInteracted.AddDynamic(this, &AMyDoor::HandleInteract);

    void AMyDoor::HandleInteract(APawn* Interactor)
    {
        OpenDoor();
    }
    ```

## Step 5: HUD Integration

Bind to `OnInteractableFound` and `OnInteractableLost` on the detector to show/hide interact prompts:

```cpp
ReachDetector->OnInteractableFound.AddDynamic(this, &AMyCharacter::ShowInteractPrompt);
ReachDetector->OnInteractableLost.AddDynamic(this, &AMyCharacter::HideInteractPrompt);
```

## Step 6: Hold Mode with Progress Bar

For a Hold interaction, bind `OnHoldProgress` on the interactable:

```cpp
Interactable->OnHoldProgress.AddDynamic(this, &AMyHUD::UpdateHoldProgress);
// Progress is 0.0 to 1.0 over HoldDuration seconds
```

## Step 7: Custom Condition (C++)

```cpp
// In your interactable subclass
virtual bool CanInteract(APawn* Interactor) const override
{
    return !bIsLocked && Interactor->HasItem(KeyItemId);
}
```
