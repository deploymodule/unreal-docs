# Tumble — Overview

## Architecture

```
UTumbleComponent  (UActorComponent, replicated)
    ├── ETumbleState         CurrentState  (Animated / Blending / Ragdoll / GetUp)
    ├── FTumbleBlendProfile  BlendProfile  — drive strengths per body region
    ├── FTumbleImpulseQueue  PendingImpulses
    └── PhysicsAnimationComponent reference
```

## States

| State | Description |
|-------|-------------|
| `Animated` | Normal animated movement, physics disabled |
| `Blending` | Physical animation drives active, gradually scaling to full ragdoll |
| `Ragdoll` | Full physics simulation, no animation drives |
| `GetUp` | Blending back from ragdoll to animation |

## Velocity-Scaled Drives

The key feature of Tumble is velocity-scaled physical animation drives. When entering ragdoll, Tumble reads the character's current velocity and scales the drive strengths inversely — a fast-moving character enters full ragdoll quickly (drives scale to zero fast), while a slow-moving or stationary character blends slowly, creating a "crumple" effect.

Drive strengths are defined per body region in `FTumbleBlendProfile`:
- **Spine** — controls torso rigidity
- **Arms** — controls arm flail
- **Legs** — controls leg collapse speed
- **Head** — controls head droop

## Impulse API

Without entering full ragdoll, you can apply world-space impulses to individual bones:

```cpp
// Bullet impact on the right shoulder
TumbleComponent->ApplyImpulse(FName("upperarm_r"), BulletDirection * 1500.f, EImpulseMode::Additive);
```

This temporarily increases the drive target offset on that bone's constraint, creating a realistic hit reaction that snaps back to the animation pose.

## Get-Up

After ragdoll, Tumble can automatically get the character back up:

1. Detect a stable resting pose (velocity below threshold)
2. Determine facing direction (face-up or face-down)
3. Teleport the capsule to match the ragdoll pelvis
4. Play the appropriate get-up animation montage
5. Blend physics drives back off as the animation takes over

## Replication

Ragdoll state transitions are replicated via RPCs. The server simulates ragdoll physics and periodically broadcasts bone transforms to clients (configurable rate via `r.Tumble.ReplicationRate`). Clients use a blended correction to avoid jarring snaps.
