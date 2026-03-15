# Lean — API Reference

## ULeanComponent

| Property | Type | Description |
|---|---|---|
| `LeanTargetComponent` | `USceneComponent*` | The component whose transform is offset by the lean (typically camera or spring arm). |
| `MaxLeanOffset` | `float` | Maximum lateral offset in world units at full lean. |
| `LeanSpeed` | `float` | Interpolation speed toward the target lean value (units/second). |
| `LeanTraceRadius` | `float` | Radius of the collision sphere trace at the lean endpoint. |
| `LeanTraceLength` | `float` | Maximum lean trace distance (should match or exceed `MaxLeanOffset`). |
| `LeanTraceChannel` | `TEnumAsByte<ECollisionChannel>` | Collision channel for lean obstruction traces. |
| `LeanInputAction` | `UInputAction*` | Optional Enhanced Input action to bind lean to automatically. |
| `bReplicateLean` | `bool` | If true, lean value is replicated to proxies. |

| Function | Signature | Description |
|---|---|---|
| `SetLeanInput` | `void (float Value)` | Sets the desired lean input in range -1.0 (left) to 1.0 (right). |
| `SetLeanDirection` | `void (FVector2D Direction)` | Sets a 2D normalized lean direction for 360-degree leaning. |
| `GetLeanValue` | `float ()` | Returns the current interpolated lean value (-1.0 to 1.0). |
| `GetLeanDirection` | `FVector2D ()` | Returns the current 2D lean direction. |
| `GetObstructedFraction` | `float ()` | Returns the collision-limited fraction (1.0 = unobstructed, <1.0 = wall nearby). |
| `IsLeaning` | `bool ()` | Returns true if the current lean value magnitude is above a small threshold (0.05). |
| `ResetLean` | `void ()` | Immediately snaps lean to zero (no interpolation). |

### Delegates

| Delegate | Parameters | Description |
|---|---|---|
| `OnLeanObstructed` | `float Fraction` | Fired when a lean collision trace hits geometry. `Fraction` is HitResult.Time. |
| `OnLeanCleared` | *(none)* | Fired when a previously obstructed lean becomes clear. |
