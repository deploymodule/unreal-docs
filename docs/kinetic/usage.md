# Kinetic — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Kinetic** — enable and restart the editor.

## Step 2: Create a Bone Mask Asset

1. Content Browser > right-click > **Miscellaneous > Data Asset > KineticBoneMaskAsset**.
2. Set **Bone Names** to the bones you want affected (e.g., `spine_01`, `spine_02`, `spine_03`).
3. Set per-bone **Weight** values (0.0–1.0).

## Step 3: Add a Kinetic Node to Your Anim Graph

### Layered Additive Apply

1. Open your Animation Blueprint.
2. In the Anim Graph right-click and search `Kinetic Additive Apply`.
3. Connect **Base Pose** and **Additive Pose** inputs.
4. Set **Bone Mask Asset** to your `UKineticBoneMaskAsset`.

### Body Dynamics

1. Search `Kinetic Body Dynamics` in the Anim Graph.
2. Connect the input pose.
3. Set **Root Bone** (e.g., `hair_root`), **Chain Length**, **Stiffness**, and **Damping**.

```
[Input Pose] → [Kinetic Body Dynamics (hair_root, length=4)] → [Output Pose]
```

## Step 4: Using from C++

```cpp
// Accessing the additive apply node from a custom AnimInstance
#include "KineticAdditiveApplyNode.h"

// In your UAnimInstance subclass
FAnimNode_KineticAdditiveApply* AdditiveNode = GetLinkedAnimGraphInstanceByTag<FAnimNode_KineticAdditiveApply>(TEXT("UpperBodyAdditive"));
if (AdditiveNode)
{
    AdditiveNode->BlendWeight = 0.75f;
}
```

## Step 5: Runtime Bone Mask Swapping

You can swap the bone mask asset at runtime to adapt between character archetypes:

```cpp
// Inside AnimInstance on character morph
KineticAdditiveNode->BoneMaskAsset = QuadrupedMaskAsset;
```

## Tips

- Keep **Chain Length** under 8 for body dynamics — longer chains increase simulation cost exponentially.
- Use **State Proxy** nodes to reduce Anim Graph visual complexity; one node replaces an entire locomotion sub-graph.
- Bone Mask Assets are reference-counted — sharing one asset across multiple nodes is safe and efficient.
