# Gimbal — Usage

## Step 1: Enable the Plugin

In the Unreal Editor go to **Edit > Plugins**, search for **Gimbal**, and enable it. Restart the editor when prompted.

## Step 2: Add UGimbalComponent to Your Player Controller

=== "Blueprint"
    1. Open your Player Controller Blueprint.
    2. In the **Components** panel click **Add** and search for `GimbalComponent`.
    3. Select the component and set **Default Mode Asset** in the Details panel.

=== "C++"
    ```cpp
    // MyPlayerController.h
    #include "Gimbal/GimbalComponent.h"

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
    TObjectPtr<UGimbalComponent> GimbalComponent;

    // MyPlayerController.cpp
    AMyPlayerController::AMyPlayerController()
    {
        GimbalComponent = CreateDefaultSubobject<UGimbalComponent>(TEXT("GimbalComponent"));
    }
    ```

## Step 3: Create a GimbalModeAsset

1. In the Content Browser right-click > **Miscellaneous > Data Asset**.
2. Choose `GimbalModeAsset` as the class.
3. Configure **Behavior Stack**, **Blend Duration**, and **Priority**.

## Step 4: Push and Pop Modes at Runtime

=== "Blueprint"
    - Call **Gimbal > Push Mode** on the Gimbal Component, passing your `GimbalModeAsset`.
    - Call **Gimbal > Pop Mode** to remove the top-most mode matching an asset reference.

=== "C++"
    ```cpp
    // Activate combat lock-on camera
    GimbalComponent->PushMode(LockOnModeAsset, EGimbalBlend::Curve);

    // Return to default after combat
    GimbalComponent->PopMode(LockOnModeAsset);
    ```

## Step 5: Configure Lock-On

```cpp
// Initiate lock-on targeting
GimbalComponent->BeginLockOn(/* optional initial target */ nullptr);

// Cycle to next valid target
GimbalComponent->CycleLockOnTarget(EGimbalCycleDir::Next);

// End lock-on
GimbalComponent->EndLockOn();
```

In Blueprint these are exposed as **Gimbal > Begin Lock On**, **Cycle Lock On Target**, and **End Lock On**.

## Step 6: Using Presets

`UGimbalPresetAsset` bundles a collection of `UGimbalModeAsset` references under a single named preset (e.g., "Combat", "Exploration", "Cinematic"). Apply a preset with:

```cpp
GimbalComponent->ApplyPreset(CombatPresetAsset);
```

This replaces the entire stack with the modes defined in the preset, blending each according to its own blend curve.

## Common Recipes

### Cinematic Fixed Shot

Create a `GimbalModeAsset` with Mode = **Fixed**, assign a world transform, push it with a high priority (e.g., 100), and pop it when the cutscene ends.

### Orbit Camera for Vehicle Inspection

Set Mode = **Orbit**, configure `MinZoom`, `MaxZoom`, `ElevationMin`, `ElevationMax`, and bind mouse wheel / right stick to `UGimbalComponent::AdjustOrbitZoom`.
