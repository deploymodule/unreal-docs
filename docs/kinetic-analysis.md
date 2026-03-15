# Kinetic Plugin — Analysis & Design Document

## What CopyMotion Does (Example Plugin Breakdown)

The example plugin provides **two runtime anim nodes** and **one animation modifier**, all focused on one core idea: extracting motion deltas from one bone and applying them to another.

### FAnimNode_CopyMotion — The Core Node

This is the main workhorse. It operates in two modes:

**Mode 1: Live Pose Delta** (`bUseBasePose = true`)
- Takes two pose inputs: a "current" pose and a "reference" pose
- Computes the delta (difference) of a source bone between the two poses
- Applies that delta to a target bone, with optional space remapping
- Supports **time delay** via Pose History (the target bone lags behind the source by N seconds)
- Supports rotation pivot points (rotate around a point, not the bone origin)

**Mode 2: Curve-Driven** (`bUseBasePose = false`)
- Reads 6 baked animation curves (TransX/Y/Z, RotRoll/Pitch/Yaw) instead of computing live deltas
- These curves are pre-baked by the LayeringMotionExtractorModifier (offline tool)
- Cheaper at runtime — no second pose evaluation needed

**Both modes share:**
- Per-axis translation scale + remap curves (UCurveVector)
- Rotation scale + remap curve (UCurveFloat)
- Translation/Rotation offset rotators (re-orient the delta before applying)
- Optional scalar curve output (extract one component of the motion as a float curve for driving other systems)
- CopySpace / ApplySpace bone references (extract delta relative to one bone, apply relative to another)

### FAnimNode_CopyBoneAdvanced — Enhanced CopyBone

An improved version of UE's built-in CopyBone:
- **Per-axis translation weights** (not just a single scalar)
- Custom bone-space axes for the weight directions (e.g., weight along the spine's forward, not world forward)
- Toggle to prevent propagation to child bones (isolates the transform change)
- Works in any bone control space (component, bone, parent, world)

### LayeringMotionExtractorModifier — Offline Curve Baking

An animation modifier that:
- Samples an animation sequence at a configurable rate
- Extracts bone motion relative to a reference bone or as component-space additive
- Bakes results into named animation curves
- Supports signal filtering (Butterworth/Chebyshev, high/low pass) to isolate frequency bands
- Post-processing: math ops, normalization, absolute value

---

## What Problem This Solves

The typical ALS (Advanced Locomotion System) workflow for upper body layering is:

1. Play locomotion animation (motion matching, blend space, etc.)
2. Layer an upper body "pose" on top (a single frame or short clip)
3. Manually set up per-bone blend weights, blend profiles, layer blend nodes
4. Hand-tune transition logic in complex state machines
5. Repeat for every stance, speed, direction combination

**Pain points:**
- State machine graphs become enormous and unmaintainable
- Per-bone blend weights are tedious to configure
- No easy way to add procedural lean, head turn, or spine twist
- Motion matching gives great lower body but upper body feels dead
- Transitions between states (stand→crouch, walk→run) require many blend nodes
- Adding "life" to upper body requires many additive animations

**CopyMotion's approach:** Instead of manually authoring additive animations, it extracts the motion delta from the locomotion itself (e.g., how the spine moves during a run cycle) and re-applies it with modifications. This means the upper body "breathing" comes from the actual locomotion, not from a separate hand-made additive.

---

## What Kinetic Should Provide

Based on the example analysis and the user's goals, Kinetic should be a **curated set of animation nodes** that simplify layered animation workflows. Organized by category:

---

### Category 1: Motion Copying & Layering (Evolved from CopyMotion)

These are the direct successors to the example nodes, modernized for UE 5.7.

#### Node: Kinetic Copy Motion
Reimplementation of FAnimNode_CopyMotion for UE 5.7:
- Copy motion delta from source bone to target bone
- Dual mode: live pose delta OR curve-driven
- Time delay via Pose History
- Space remapping (extract in one bone's space, apply in another's)
- Per-axis scale, remap curves, rotation pivot
- Scalar curve output for driving other systems
- **Changes from example:** Remove dependency on PoseSearch module (use our own lightweight history buffer), clean API naming

#### Node: Kinetic Copy Bone (Advanced)
Reimplementation of FAnimNode_CopyBoneAdvanced:
- Per-axis translation weights with custom weight axes
- Propagation control (isolate from children)
- Multi-space operation

#### Tool: Kinetic Motion Extractor (Animation Modifier)
Reimplementation of LayeringMotionExtractorModifier:
- Offline curve baking from bone motion
- Signal filtering (isolate frequency bands — e.g., high-freq jitter vs. low-freq sway)
- Normalization and math post-processing

---

### Category 2: Procedural Body Dynamics

New nodes that generate motion procedurally from gameplay data, no animations needed.

#### Node: Kinetic Lean
Applies procedural lean to spine/pelvis based on velocity and acceleration:
- **Input:** Character velocity, acceleration (or auto-read from movement component)
- **Output:** Rotation applied to one or more bones (spine chain or pelvis)
- **Parameters:** Lean angle limits, spring damping, response speed, axis mapping
- **Algorithm:** Project acceleration onto the character's lateral axis → spring-damped rotation
- **Use case:** Character leans into turns, leans back when braking, leans forward when accelerating
- No animation assets required — pure math

#### Node: Kinetic Look At
Procedural head/neck tracking toward a world target:
- **Input:** World-space target location (or actor reference)
- **Output:** Rotation applied to head and optionally neck bones
- **Parameters:** Max rotation angles (yaw/pitch), interpolation speed, dead zone, bone chain distribution
- **Algorithm:** IK-style chain solve distributing rotation across N bones with configurable weights
- **Clamp modes:** Hard clamp, soft falloff, body-turn trigger threshold
- **Use case:** Character looks at interactables, NPCs, threats — without needing aim offset animations

#### Node: Kinetic Aim Offset (Procedural)
Procedural spine + upper body rotation for aiming:
- **Input:** Aim direction (world or local space)
- **Output:** Rotation distributed across spine chain
- **Parameters:** Per-bone weight distribution, twist vs. bend ratio, clamp angles, interpolation
- **Algorithm:** Distributes aim rotation across spine_01 through spine_05 + head, with configurable weight curve
- **Difference from Look At:** Affects the full spine chain for weapon aiming, not just head/neck for looking
- **Use case:** Replaces traditional aim offset blend spaces with a single procedural node. Works with ANY locomotion animation.

#### Node: Kinetic Spine Twist
Isolated upper body twist (yaw rotation distributed along spine):
- **Input:** Twist angle (float)
- **Output:** Yaw rotation distributed across specified bone chain
- **Parameters:** Bone chain, per-bone weight curve, max twist per bone, twist propagation direction
- **Use case:** Upper body faces different direction than lower body (strafing, aiming sideways while running)

#### Node: Kinetic Breathing
Subtle procedural breathing motion:
- **Input:** Breathing rate, depth, stress multiplier
- **Output:** Translation + rotation on spine/chest bones
- **Algorithm:** Sine wave with configurable frequency, amplitude, and per-bone weights. Stress parameter increases rate and amplitude.
- **Use case:** Idle breathing, heavy breathing after sprinting, stressed breathing in combat. No animation assets needed.

---

### Category 3: Locomotion State Helpers

Nodes that simplify common state machine patterns into single-node solutions.

#### Node: Kinetic Stance Blend
Simplified stand ↔ crouch (↔ prone) blending:
- **Input:** Stance enum (Standing, Crouching, Prone) or float 0→1→2
- **Output:** Blended pose
- **Pose inputs:** Standing pose, Crouching pose, optional Prone pose
- **Parameters:** Blend time, blend curve, per-bone blend profile
- **What it replaces:** A multi-state blend node + transition rules + blend profile setup. One node instead of a mini state machine.

#### Node: Kinetic Gait Blend
Simplified idle → walk → run → sprint blending:
- **Input:** Speed (float, auto-read from movement component or manual)
- **Output:** Blended pose
- **Pose inputs:** Idle, Walk, Run, Sprint (2-4 inputs, unused ones skip)
- **Parameters:** Speed thresholds per transition, blend ranges, blend curves
- **What it replaces:** A Blend Space 1D with manual axis setup, or a 4-state machine with speed-based transitions.
- **Bonus:** Auto-detects direction for blend space variant (forward/backward/strafe)

#### Node: Kinetic State Blend
General-purpose multi-state blender:
- **Input:** Active state index (int) or enum
- **Output:** Blended pose
- **Pose inputs:** Up to N poses (dynamic pin count)
- **Parameters:** Per-transition blend time, blend curve, optional cross-fade vs. frozen-pose transition
- **What it replaces:** A full state machine graph when the transition logic is simple (enum-driven, no complex conditions)

---

### Category 4: Layer Blend Utilities

Nodes that make per-bone layering easier.

#### Node: Kinetic Layer Blend
Simplified layered blend per bone:
- **Input:** Base pose, Layer pose
- **Parameters:** Blend profile reference, alpha, mask preset (UpperBody, Arms, LeftArm, RightArm, Head, Spine, Custom)
- **Presets auto-configure** the blend weights for common bone sets. No manual per-bone weight painting for standard use cases.
- **What it replaces:** LayeredBlendPerBone with manual weight setup for every skeleton

#### Node: Kinetic Additive Apply
Simplified additive pose application:
- **Input:** Base pose, Additive pose (single frame or sequence)
- **Parameters:** Alpha, per-bone weight mask (preset or custom)
- **What it replaces:** ApplyAdditive + manual blend profile. Common for upper body overlays.

---

### Category 5: Utility Nodes

Small helpers that come up constantly in animation graphs.

#### Node: Kinetic Bone Transform
Set/modify a single bone's transform directly:
- **Input:** Bone name, translation/rotation/scale deltas or absolute values
- **Parameters:** Space (local, component, world), blend alpha, additive vs. override mode
- **What it replaces:** ModifyBone node but with a cleaner API and better defaults

#### Node: Kinetic Speed Warp
Adjusts animation playback to match actual movement speed:
- **Input:** Ground speed, animation speed
- **Output:** Play rate multiplier + stride scale
- **Parameters:** Min/max playback rate, stride scaling curve
- **Use case:** Prevents foot sliding when actual speed doesn't match animation speed

#### Node: Kinetic Foot Lock (Simple)
Basic foot planting without full IK:
- **Input:** Base pose
- **Parameters:** Foot bones, lock threshold (speed), blend in/out times
- **Algorithm:** When foot velocity drops below threshold, locks the foot position in world space. Blends smoothly in/out.
- **Use case:** Prevents foot sliding during turns, stops, and start-ups

---

## Module Architecture

```
Plugins/Kinetic/
  Kinetic.uplugin
  Shaders/                              (empty for now, may need compute for batched solves)
  Source/
    Kinetic/                            (Runtime module)
      Kinetic.Build.cs
      Kinetic.h / .cpp                  (Module boilerplate)
      Public/
        AnimNodes/
          AnimNode_KineticCopyMotion.h
          AnimNode_KineticCopyBone.h
          AnimNode_KineticLean.h
          AnimNode_KineticLookAt.h
          AnimNode_KineticAimOffset.h
          AnimNode_KineticSpineTwist.h
          AnimNode_KineticBreathing.h
          AnimNode_KineticStanceBlend.h
          AnimNode_KineticGaitBlend.h
          AnimNode_KineticStateBlend.h
          AnimNode_KineticLayerBlend.h
          AnimNode_KineticAdditiveApply.h
          AnimNode_KineticBoneTransform.h
          AnimNode_KineticSpeedWarp.h
          AnimNode_KineticFootLock.h
        KineticTypes.h                  (Shared enums, structs)
      Private/
        AnimNodes/
          AnimNode_KineticCopyMotion.cpp
          AnimNode_KineticCopyBone.cpp
          AnimNode_KineticLean.cpp
          AnimNode_KineticLookAt.cpp
          AnimNode_KineticAimOffset.cpp
          AnimNode_KineticSpineTwist.cpp
          AnimNode_KineticBreathing.cpp
          AnimNode_KineticStanceBlend.cpp
          AnimNode_KineticGaitBlend.cpp
          AnimNode_KineticStateBlend.cpp
          AnimNode_KineticLayerBlend.cpp
          AnimNode_KineticAdditiveApply.cpp
          AnimNode_KineticBoneTransform.cpp
          AnimNode_KineticSpeedWarp.cpp
          AnimNode_KineticFootLock.cpp
    KineticEditor/                      (Editor module)
      KineticEditor.Build.cs
      KineticEditor.h / .cpp
      Public/
        AnimGraphNodes/
          AnimGraphNode_KineticCopyMotion.h
          AnimGraphNode_KineticCopyBone.h
          AnimGraphNode_KineticLean.h
          AnimGraphNode_KineticLookAt.h
          AnimGraphNode_KineticAimOffset.h
          AnimGraphNode_KineticSpineTwist.h
          AnimGraphNode_KineticBreathing.h
          AnimGraphNode_KineticStanceBlend.h
          AnimGraphNode_KineticGaitBlend.h
          AnimGraphNode_KineticStateBlend.h
          AnimGraphNode_KineticLayerBlend.h
          AnimGraphNode_KineticAdditiveApply.h
          AnimGraphNode_KineticBoneTransform.h
          AnimGraphNode_KineticSpeedWarp.h
          AnimGraphNode_KineticFootLock.h
        AnimationModifiers/
          KineticMotionExtractorModifier.h
      Private/
        AnimGraphNodes/
          AnimGraphNode_KineticCopyMotion.cpp
          ... (one per node)
        AnimationModifiers/
          KineticMotionExtractorModifier.cpp
```

### Build.cs Dependencies

**Runtime (Kinetic):**
- `Core`, `CoreUObject`, `Engine`
- `AnimGraphRuntime` — for `FAnimNode_SkeletalControlBase` and related base classes
- `AnimationCore` — for bone references, compact pose types

**Editor (KineticEditor):**
- `Core`, `CoreUObject`, `Engine`, `UnrealEd`
- `AnimGraph` — for `UAnimGraphNode_SkeletalControlBase`
- `BlueprintGraph` — for pin/detail customization
- `Kinetic` (the runtime module)
- `AnimationModifierLibrary` — only if we reuse Epic's motion extractor utilities (optional)

### No PoseSearch Dependency

The CopyMotion example depends on the `PoseSearch` module for its Pose History time-delay feature. We should implement our own lightweight ring buffer for delayed bone transforms instead, since:
- PoseSearch is a heavyweight module primarily for motion matching
- Not all projects use motion matching
- A simple ring buffer of bone transforms at timestamped intervals is ~50 lines of code
- Removes a coupling that would force users to enable the PoseSearch plugin

---

## Implementation Priority

### Phase 1 — Core Motion Nodes (Direct value, proven patterns from CopyMotion)
1. Kinetic Copy Motion
2. Kinetic Copy Bone (Advanced)
3. Kinetic Motion Extractor Modifier
4. Kinetic Layer Blend (preset-based)
5. Kinetic Bone Transform

### Phase 2 — Procedural Body Dynamics (Biggest UX wins, no animation assets needed)
6. Kinetic Lean
7. Kinetic Look At
8. Kinetic Aim Offset
9. Kinetic Spine Twist
10. Kinetic Breathing

### Phase 3 — State Simplification (Replaces complex state machines)
11. Kinetic Stance Blend
12. Kinetic Gait Blend
13. Kinetic State Blend
14. Kinetic Additive Apply

### Phase 4 — Polish & Advanced
15. Kinetic Speed Warp
16. Kinetic Foot Lock

---

## Key Technical Decisions

### Base Classes
- **Skeletal control nodes** (single bone manipulation): inherit `FAnimNode_SkeletalControlBase`
- **Multi-pose blend nodes** (stance blend, gait blend, state blend): inherit `FAnimNode_Base` directly, manage multiple `FPoseLink` inputs
- **Layer blend nodes**: may inherit `FAnimNode_LayeredBoneBlend` or reimplement for preset support

### Thread Safety
All `EvaluateSkeletalControl_AnyThread` and `Evaluate_AnyThread` implementations must be fully thread-safe:
- No UObject access
- No heap allocation
- Pre-cache all bone indices in `InitializeBoneReferences` / `CacheBones`
- Use `FAnimWeight::IsRelevant()` for early-outs

### Animation Graph Node Editor UX
Each editor node should provide:
- Descriptive dynamic titles (show bone names, state names)
- Sensible defaults (works with zero configuration on Mannequin skeleton)
- Pin exposure for commonly animated properties (alpha, blend weights, target location)
- Tooltip documentation on every property
- Category organization: all nodes under "Kinetic" with subcategories

### Skeleton Agnostic
Nodes should work with ANY skeleton, not just UE5 Mannequin:
- Bone names are always user-configurable (FBoneReference)
- Presets (UpperBody, etc.) use convention-based bone detection with fallbacks
- No hardcoded bone names except as defaults

---

## Summary

Kinetic fills the gap between "raw animation nodes" and "full locomotion system" (like ALS). It gives users:

1. **Motion Layering** — Extract and re-apply motion deltas without manual additive animation authoring
2. **Procedural Dynamics** — Lean, look-at, aim, breathing with zero animation assets
3. **State Simplification** — Replace 20-node state machines with single blend nodes
4. **Layer Presets** — Per-bone blending without manual weight painting

The plugin targets users who have locomotion working (motion matching, blend spaces, whatever) and want to quickly add upper body life, procedural reactions, and cleaner graph organization without building ALS-scale complexity.
