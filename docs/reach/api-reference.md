# Reach — API Reference

## UReachDetectorComponent

| Property | Type | Description |
|---|---|---|
| `DetectionRadius` | `float` | Sphere sweep radius for finding interactables. |
| `DetectionInterval` | `float` | Seconds between detection sweeps. |
| `DetectionChannel` | `TEnumAsByte<ECollisionChannel>` | Collision channel used for detection sweeps. |
| `InteractInputAction` | `UInputAction*` | Optional Enhanced Input action to bind interaction to. |
| `MaxInteractables` | `int32` | Maximum number of interactables tracked simultaneously. |

| Function | Signature | Description |
|---|---|---|
| `TriggerInteract` | `void ()` | Manually triggers an interaction on the current focused interactable. |
| `GetFocusedInteractable` | `UReachInteractableComponent* ()` | Returns the highest-priority in-range interactable. |
| `GetAllInRangeInteractables` | `TArray<UReachInteractableComponent*> ()` | Returns all currently detected interactables. |
| `SetDetectionEnabled` | `void (bool)` | Enables or disables detection sweeps. |

### Delegates

| Delegate | Parameters | Description |
|---|---|---|
| `OnInteractableFound` | `UReachInteractableComponent*` | Fired when a new interactable enters detection range. |
| `OnInteractableLost` | `UReachInteractableComponent*` | Fired when an interactable leaves detection range. |
| `OnFocusChanged` | `UReachInteractableComponent*` | Fired when the focused (highest-priority) interactable changes. |

## UReachInteractableComponent

| Property | Type | Description |
|---|---|---|
| `InteractionMode` | `EReachInteractionMode` | The interaction mode for this interactable. |
| `DisplayName` | `FText` | Name shown in HUD prompts. |
| `Priority` | `int32` | Higher values take focus when multiple interactables overlap. |
| `bReplicateInteraction` | `bool` | If true, interaction events are server-validated via RPC. |
| `HoldDuration` | `float` | Required hold time in seconds (Hold mode only). |
| `SmashCount` | `int32` | Required presses (Smash mode only). |
| `SmashWindow` | `float` | Time window in seconds for smash inputs. |
| `RhythmCurve` | `UCurveFloat*` | Timing pattern curve (Rhythm mode only). |

| Function | Signature | Description |
|---|---|---|
| `CanInteract` | `virtual bool (APawn*)` | Override to add custom interaction conditions. |
| `SetEnabled` | `void (bool)` | Enables or disables this interactable at runtime. |

### Delegates

| Delegate | Parameters | Description |
|---|---|---|
| `OnInteracted` | `APawn* Interactor` | Fired when interaction completes successfully. |
| `OnHoldProgress` | `float Progress` | Fired each frame during Hold mode with 0.0–1.0 progress. |
| `OnHoldCancelled` | *(none)* | Fired if Hold interaction is cancelled before completion. |
| `OnSmashProgress` | `int32 CurrentCount` | Fired after each valid smash press. |

## EReachInteractionMode

| Value | Description |
|---|---|
| `Instant` | Single press. |
| `Hold` | Sustained press for `HoldDuration`. |
| `Smash` | Repeated presses within `SmashWindow`. |
| `Rhythm` | Timed presses following `RhythmCurve` pattern. |
