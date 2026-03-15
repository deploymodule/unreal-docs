# Warden — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Warden** — enable and restart.

## Step 2: Create a StateTree Asset

1. Content Browser > right-click > **Artificial Intelligence > State Tree**.
2. Open the State Tree Editor.
3. In the **Schema** dropdown select `WardenStateTreeSchema` (or the default AI schema).

## Step 3: Add Warden Tasks to States

1. Select a state in the State Tree Editor.
2. Click **Add Task**.
3. Search for any Warden task (e.g., `Warden: Move To Target`, `Warden: Attack Melee`).
4. Configure task properties in the Details panel.

### Example: Simple Combat State

```
[Root]
 ├── [Idle]    → Task: Warden_WaitDelay (3.0s)
 ├── [Alert]   → Task: Warden_MoveToLastKnownPosition
 └── [Combat]
      ├── Task: Warden_AttackMelee
      └── Task: Warden_Strafe
```

## Step 4: Add Warden Conditions

In a state's **Enter Conditions** list, add any `FWardenCond_*` condition:

```
[Combat] Enter Conditions:
  - FWardenCond_IsTargetAlive  (Target = Blackboard.TargetActor)
  - FWardenCond_IsTargetInRange  (Range = 300.0)
```

## Step 5: Set Up the Memory Layer

Add a `UWardenMemoryComponent` to your AI Character:

=== "C++"
    ```cpp
    #include "Warden/WardenMemoryComponent.h"

    UPROPERTY(VisibleAnywhere)
    TObjectPtr<UWardenMemoryComponent> WardenMemory;

    // Constructor
    WardenMemory = CreateDefaultSubobject<UWardenMemoryComponent>(TEXT("WardenMemory"));
    WardenMemory->MemoryDecayRate = 0.1f;  // confidence units per second
    WardenMemory->MaxMemoryAge = 60.f;     // forget after 60 seconds
    ```

## Step 6: Use EQS Templates

1. In any task that has an **EQS Query** pin (e.g., `Warden_FindCover`), click the pin.
2. Select one of the five Warden EQS template assets from the dropdown.
3. The task will run the query at the appropriate moment and store the result in the Blackboard.

## Customizing Tasks (C++)

All Warden tasks derive from `FWardenTask_Base`. Override `EnterState`, `TickTask`, and `ExitState`:

```cpp
#include "Warden/WardenTaskBase.h"

USTRUCT()
struct FMyCustomTask : public FWardenTask_Base
{
    GENERATED_BODY()

    virtual EStateTreeRunStatus EnterState(FStateTreeExecutionContext& Context,
        const FStateTreeTransitionResult& Transition) override;

    virtual EStateTreeRunStatus TickTask(FStateTreeExecutionContext& Context,
        const float DeltaTime) override;
};
```
