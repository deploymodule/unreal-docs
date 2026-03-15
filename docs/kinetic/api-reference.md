# Kinetic — API Reference

## FAnimNode_KineticAdditiveApply

Layers an additive pose over a base pose using a configurable bone mask.

| Property | Type | Description |
|---|---|---|
| `BasePose` | `FPoseLink` | The base input pose to layer onto. |
| `AdditivePose` | `FPoseLink` | The additive pose to apply. |
| `BoneMaskAsset` | `UKineticBoneMaskAsset*` | Bone mask defining which bones receive the additive blend and at what weight. |
| `BlendWeight` | `float` | Global multiplier (0.0–1.0) applied on top of per-bone weights. |
| `bMeshSpaceAdditive` | `bool` | If true, applies the additive in mesh space rather than local space. |

## FAnimNode_KineticBodyDynamics

Spring-physics secondary motion for bone chains. Runs on the animation worker thread.

| Property | Type | Description |
|---|---|---|
| `ComponentPose` | `FPoseLink` | Input pose to apply dynamics on top of. |
| `RootBone` | `FBoneReference` | First bone in the simulated chain. |
| `ChainLength` | `int32` | Number of bones to simulate from the root. |
| `Stiffness` | `float` | Spring stiffness. Higher values = stiffer, less motion. |
| `Damping` | `float` | Damping ratio. Higher values = motion dies faster. |
| `GravityScale` | `float` | Multiplier on world gravity applied to each bone in the chain. |
| `MaxAngle` | `float` | Maximum deflection angle in degrees from rest pose. |

## FAnimNode_KineticStateProxy

Wraps multiple Blend Spaces behind a single normalized direction input.

| Property | Type | Description |
|---|---|---|
| `DirectionVector` | `FVector2D` | Normalized cardinal direction (forward/back/left/right). |
| `BlendSpaces` | `TArray<UBlendSpace*>` | Up to 8 blend spaces mapped to cardinal directions. |
| `TransitionTime` | `float` | Cross-fade time in seconds when switching between blend spaces. |

## UKineticBoneMaskAsset

| Property | Type | Description |
|---|---|---|
| `MaskName` | `FName` | Human-readable identifier for this mask. |
| `BoneEntries` | `TArray<FKineticBoneEntry>` | Per-bone blend weight entries. |

## FKineticBoneEntry (Struct)

| Field | Type | Description |
|---|---|---|
| `BoneName` | `FName` | Skeleton bone name. |
| `Weight` | `float` | Blend weight for this bone (0.0–1.0). |
| `bIncludeChildren` | `bool` | If true, all child bones inherit this entry's weight unless overridden. |
