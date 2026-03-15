# Lean — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Lean** — enable and restart.

## Step 2: Add ULeanComponent to Your Character

=== "Blueprint"
    1. Open your Character Blueprint.
    2. **Add Component > LeanComponent**.
    3. Set **Lean Target Component** to your camera or spring arm in Details.

=== "C++"
    ```cpp
    #include "Lean/LeanComponent.h"

    UPROPERTY(VisibleAnywhere)
    TObjectPtr<ULeanComponent> LeanComp;

    // Constructor
    LeanComp = CreateDefaultSubobject<ULeanComponent>(TEXT("LeanComponent"));
    LeanComp->LeanTargetComponent = CameraComponent;
    LeanComp->MaxLeanOffset = 60.f;
    LeanComp->LeanSpeed = 8.f;
    ```

## Step 3: Drive Lean from Input

=== "Enhanced Input (Recommended)"
    1. Create an `InputAction` for lean (Axis1D or Axis2D).
    2. In Details on `ULeanComponent`, assign **Lean Input Action**.
    3. Lean drives itself — no event graph wiring needed.

=== "Manual C++"
    ```cpp
    // In your Character or Controller's input handler
    void AMyCharacter::OnLeanInput(float Value)
    {
        LeanComp->SetLeanInput(Value); // -1.0 = full left, 1.0 = full right
    }
    ```

=== "Blueprint"
    - Call **Lean > Set Lean Input** from your input event with the axis value.

## Step 4: Connect Lean to Animation

In your Animation Blueprint:

1. Add a `float` variable `LeanValue`.
2. In `Event Blueprint Update Animation`, call **Lean > Get Lean Value** on the owning character's Lean Component and store the result.
3. Use `LeanValue` to drive a `Lean_LeftRight` blend space or a morph target.

```
[BlueprintUpdateAnimation] → [Get Lean Component] → [Get Lean Value] → [Set LeanValue]
```

## Step 5: Tune Collision

```cpp
LeanComp->LeanTraceRadius = 20.f;   // sphere size at lean target position
LeanComp->LeanTraceLength = 80.f;   // max lean distance in world units
LeanComp->LeanTraceChannel = ECC_Camera;
```

## Step 6: Cover-Peek Preset

For a tactical cover-peek setup:
- Set `MaxLeanOffset` to the distance from cover to the peek position.
- Bind a `IA_LeanRight` and `IA_LeanLeft` action to `SetLeanInput(1.0)` and `SetLeanInput(-1.0)` respectively.
- Set `LeanSpeed` to `10.0` for a snappy feel or `4.0` for a slower cinematic lean.
