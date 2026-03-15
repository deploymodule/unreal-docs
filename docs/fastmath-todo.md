# FastMath — Blueprint Node Plan

## Plugin Overview

**Plugin name:** `FastMath`
**Class:** `UFastMathFunctionLibrary`
**Category root:** `"Fast Math"`
**uplugin FriendlyName:** `"Fast Math"`
**Description:** High-performance Blueprint math nodes for game logic. Replaces node-heavy patterns
with single C++ calls: bitmask state, noise, trajectory prediction, grid math, spring dynamics,
easing, Bezier curves, weighted random, and more.

## Conventions (matching FastFind / FastLoop)

- All public functions prefixed `FastMath` — e.g. `FastMathSpringFloat`
- Subcategories use `|` separator — e.g. `"Fast Math|Noise"`
- Output params prefixed `Out`, booleans prefixed `b`
- `UPARAM(ref)` only where the caller owns mutable state (spring velocity, moving-avg history)
- Private helpers at the bottom; no UFUNCTION on them
- Section banners: `// ========== SECTION ==========`

---

## Nodes

### 1. BITMASK — `"Fast Math|Bitmask"`

Pack up to 32 boolean states (Stunned, Sprinting, Poisoned…) into one `int32`.
Read/write in a single CPU cycle with no map lookups.

| Function | Signature | Notes |
|---|---|---|
| `FastMathSetBit` | `(Mask, BitIndex, bValue) → int32` | Sets or clears one bit |
| `FastMathGetBit` | `(Mask, BitIndex) → bool` | Reads one bit |
| `FastMathToggleBit` | `(Mask, BitIndex) → int32` | XOR flip |
| `FastMathCountBits` | `(Mask) → int32` | Popcount — how many flags are active |
| `FastMathHasAllBits` | `(Mask, Required) → bool` | `(Mask & Required) == Required` |
| `FastMathHasAnyBit` | `(Mask, Required) → bool` | `(Mask & Required) != 0` |
| `FastMathClearBits` | `(Mask, ToClear) → int32` | Clears a set of bits in one call |

---

### 2. NOISE — `"Fast Math|Noise"`

Gradient value noise (Perlin-style, no patent issues). Seed-stable across frames.
No external library — self-contained permutation table baked into .cpp.

| Function | Signature | Notes |
|---|---|---|
| `FastMathNoise1D` | `(X, Frequency, Seed) → float` | Returns [-1, 1]. Good for 1D camera shake |
| `FastMathNoise2D` | `(X, Y, Frequency, Seed) → float` | 2D surface noise |
| `FastMathNoise3D` | `(X, Y, Z, Frequency, Seed) → float` | 3D volumetric noise |
| `FastMathFractalNoise2D` | `(X, Y, Octaves, Frequency, Lacunarity, Persistence, Seed) → float` | fBm — layered octaves for terrain/cloud detail |

**Decisions:**
- Use a fixed 256-entry permutation table; seed shuffles it via a fast hash
- Return range normalized to [-1, 1] for all variants
- `FractalNoise2D` only (3D fBm is rarely needed; add later if requested)

---

### 3. TRAJECTORY — `"Fast Math|Trajectory"`

Pure kinematic math. No physics engine, no prediction subsystem.

| Function | Signature | Notes |
|---|---|---|
| `FastMathProjectilePosition` | `(Start, LaunchVelocity, Time, GravityZ) → FVector` | P = P₀ + V·t + ½g·t². Single point query |
| `FastMathProjectileArc` | `(Start, LaunchVelocity, GravityZ, TimeStep, NumSteps, OutPoints)` | Fills arc path array for a trajectory preview line |
| `FastMathLaunchVelocityToTarget` | `(Start, Target, LaunchAngleDeg, GravityZ, OutVelocity) → bool` | Solves launch speed for a fixed angle; returns false if target is out of range |

**Decisions:**
- `GravityZ` is negative in UE world space — let caller pass whatever their gravity is
- `ProjectileArc` pre-allocates `OutPoints` to `NumSteps` to avoid repeated realloc
- No drag/air resistance — intentionally simple; drag belongs in a physics plugin

---

### 4. GRID SNAP — `"Fast Math|Grid"`

Lightning-fast coordinate conversions. No division nodes required.

| Function | Signature | Notes |
|---|---|---|
| `FastMathSnapToGrid` | `(Location, CellSize) → FVector` | Square grid snap via `round(v/s)*s` |
| `FastMathSnapToHexGrid` | `(Location, HexRadius) → FVector` | Axial hex coordinate round-trip — correct cube rounding |
| `FastMathWorldToCell` | `(Location, GridOrigin, CellSize) → FIntPoint` | World pos → 2D grid cell index |
| `FastMathCellToWorld` | `(Cell, GridOrigin, CellSize) → FVector` | Reverse; returns cell center |

**Decisions:**
- Hex uses pointy-top orientation (most common for strategy games)
- `WorldToCell` floors to cell, not rounds — consistent with "which cell am I in?"
- Z is preserved in `SnapToGrid` (snaps XY only); add a `bSnapZ` param

---

### 5. SPIN DEFLECT — `"Fast Math|Physics"`

Physically-motivated bounce vectors without the physics engine.

| Function | Signature | Notes |
|---|---|---|
| `FastMathReflect` | `(Velocity, Normal, Elasticity, Friction) → FVector` | Standard reflect with energy loss (Elasticity) and tangential damping (Friction). Most use cases. |
| `FastMathSpinDeflect` | `(Velocity, Normal, SpinAxis, SpinStrength, Friction) → FVector` | Adds angular bias to the bounce direction based on object spin. For billiards, curved projectiles, English-ball physics. |

**Decisions:**
- `FastMathReflect` covers 90% of use cases cleanly; `SpinDeflect` is opt-in complexity
- SpinAxis is the angular velocity vector of the bouncing object (normalized internally)
- Friction [0,1]: 0 = frictionless slide, 1 = full stop tangentially

---

### 6. WRAP — `"Fast Math|Wrap"`

Modulo-based coordinate wrapping. No branches, no jitter.

| Function | Signature | Notes |
|---|---|---|
| `FastMathWrapFloat` | `(Value, Min, Max) → float` | Scalar modulo wrap — underlying all other wrap nodes |
| `FastMathWrapLocation` | `(Location, MinBounds, MaxBounds) → FVector` | Full 3D torus wrap |
| `FastMathWrapLocation2D` | `(Location, MinBounds, MaxBounds) → FVector` | XY wrap only; Z passed through |

**Decisions:**
- Uses `FMath::Fmod`-based formula that handles negatives correctly (unlike plain `%`)
- 2D and 3D variants are separate nodes for clarity — most screen-wrap use cases are 2D

---

### 7. SPRING — `"Fast Math|Spring"`

Critically-damped spring integration. The "game feel" node.

| Function | Signature | Notes |
|---|---|---|
| `FastMathSpringFloat` | `(Current, Target, UPARAM(ref) Velocity, Stiffness, Damping, DeltaTime, OutVelocity) → float` | Analytically integrated spring — frame-rate independent |
| `FastMathSpringVector` | `(Current, Target, UPARAM(ref) Velocity, Stiffness, Damping, DeltaTime, OutVelocity) → FVector` | Same, per-component on a vector |

**Decisions:**
- Caller owns `Velocity` as a Blueprint variable and feeds it back each frame
- Analytical integration (not Euler) so it's stable at large DeltaTime and high Stiffness
- Critical damping occurs when `Damping = 2 * sqrt(Stiffness)` — worth a comment in the tooltip
- No `FSpringInterpTo` wrapper — just the raw math, callers can wrap it themselves

---

### 8. MORTON CODE — `"Fast Math|Morton"`

Z-order curve encoding. Converts 2D/3D grid coordinates to a single integer for O(1) grid neighbor lookup.

| Function | Signature | Notes |
|---|---|---|
| `FastMathEncodeMorton2D` | `(X, Y) → int32` | Interleaves 16-bit X/Y into 32 bits |
| `FastMathDecodeMorton2D` | `(MortonCode, OutX, OutY)` | Reverse |
| `FastMathEncodeMorton3D` | `(X, Y, Z) → int64` | Interleaves 21-bit X/Y/Z into 63 bits |
| `FastMathDecodeMorton3D` | `(MortonCode, OutX, OutY, OutZ)` | Reverse |

**Decisions:**
- Use bit-spread magic number trick (no loops) — truly single-cycle
- int32 for 2D (16 bits per axis, supports 65536² grids), int64 for 3D
- No `MortonNeighbors` node — caller adds/subtracts Morton-space offsets themselves; document this in tooltip

---

### 9. WAVE — `"Fast Math|Wave"`

Oscillator math. Replaces tick-event node clusters for hovering, breathing, and bobbing.

| Function | Signature | Notes |
|---|---|---|
| `FastMathWave` | `(Time, Frequency, Amplitude, Phase) → float` | Simple sine wave. Phase in degrees. |
| `FastMathCompoundWave` | `(Time, SineFreq, CosFreq, SineAmp, CosAmp, Offset) → float` | Blends sine and cosine — produces non-repeating organic motion |
| `FastMathPulse` | `(Time, Frequency, Sharpness) → float` | `pow(sin(t), Sharpness)` — smooth at 1.0, heartbeat-like at high values |

**Decisions:**
- All inputs in real-world units (seconds, Hz) not radians — BP-friendly
- `CompoundWave` is the "spooky hover" node from the pitch — two oscillators in one
- `Pulse` is not in the original list but replaces ~4 nodes (sine → abs → power → remap) for breathing FX

---

### 10. WEIGHTED RANDOM — `"Fast Math|Random"`

Loot tables and AI decision rolls in one node.

| Function | Signature | Notes |
|---|---|---|
| `FastMathWeightedRandom` | `(Weights) → int32` | Cumulative-sum roll. Returns winning index. |
| `FastMathWeightedRandomSeeded` | `(Weights, Seed) → int32` | Deterministic version — same Seed always returns same index for same Weights |

**Decisions:**
- Weights are `TArray<float>` — raw unnormalized values. No need to sum to 100 or 1.0.
- Returns -1 if Weights is empty
- Seeded variant uses a fast LCG from Seed, not `FMath::RandRange` — fully reproducible

---

### 11. EASE — `"Fast Math|Ease"`

Professional easing equations without Timelines or curve assets.

**Enum:** `EFastMathEase`
```
Linear,
SineIn, SineOut, SineInOut,
QuadIn, QuadOut, QuadInOut,
CubicIn, CubicOut, CubicInOut,
QuartIn, QuartOut, QuartInOut,
ExpoIn, ExpoOut, ExpoInOut,
CircIn, CircOut, CircInOut,
BackIn, BackOut, BackInOut,
ElasticIn, ElasticOut, ElasticInOut,
BounceIn, BounceOut, BounceInOut
```

| Function | Signature | Notes |
|---|---|---|
| `FastMathEaseAlpha` | `(Alpha, Type) → float` | Core: applies easing to a [0,1] alpha. Use this + any Lerp. |
| `FastMathEaseFloat` | `(A, B, Alpha, Type) → float` | Eased lerp between two floats |
| `FastMathEaseVector` | `(A, B, Alpha, Type) → FVector` | Eased lerp between two vectors |

**Decisions:**
- `EaseAlpha` is the core — other two are convenience wrappers
- Back/Elastic/Bounce require no extra parameters (built-in sane defaults for overshoot/period)
- 30 enum entries is a lot but is standard for this category — it's what devs expect

---

### 12. MOVING AVERAGE / LOW-PASS — `"Fast Math|Smooth"`

Filter jitter from raw data streams.

| Function | Signature | Notes |
|---|---|---|
| `FastMathLowPassFloat` | `(NewValue, Current, Smoothing) → float` | Exponential moving average. O(1), no history. `Smoothing` [0,1]: 0=instant, 1=frozen. |
| `FastMathLowPassVector` | `(NewValue, Current, Smoothing) → FVector` | Same, per-component |
| `FastMathMovingAvgFloat` | `(NewValue, UPARAM(ref) History, SampleCount, OutHistory) → float` | True N-sample circular buffer average. Caller stores History array. |
| `FastMathMovingAvgVector` | `(NewValue, UPARAM(ref) History, SampleCount, OutHistory) → FVector` | Same for vectors |

**Decisions:**
- LowPass covers 90% of smoothing needs with zero Blueprint variables (just feed back the output)
- MovingAvg is for cases where you need a true N-frame window (VR hand tracking, custom cameras)
- `SampleCount` in MovingAvg caps the buffer — History is trimmed to SampleCount internally

---

### 13. SAFE ROTATION — `"Fast Math|Rotation"`

Quaternion math surfaced as Rotator nodes. Kills Gimbal Lock.

| Function | Signature | Notes |
|---|---|---|
| `FastMathCombineRotators` | `(A, B) → FRotator` | Quat multiply → FRotator. No Gimbal Lock. |
| `FastMathSlerpRotators` | `(A, B, Alpha) → FRotator` | Shortest-path spherical interpolation |
| `FastMathAngleBetweenRotators` | `(A, B) → float` | Angular distance in degrees (always positive) |
| `FastMathAngleBetweenVectors` | `(A, B) → float` | Simpler: angle between two direction vectors |

**Decisions:**
- All functions accept and return FRotator (what BP devs use) — quat is internal only
- `SlerpRotators` takes the shortest arc by default (negates quat if dot < 0)
- `AngleBetweenVectors` normalizes internally — caller does not need to normalize

---

### 14. BEZIER — `"Fast Math|Bezier"`

Pure math splines. No SplineComponent required.

| Function | Signature | Notes |
|---|---|---|
| `FastMathEvalQuadraticBezier` | `(P0, P1, P2, Alpha) → FVector` | 3-point. Simple arc — good for simple projectiles |
| `FastMathEvalCubicBezier` | `(P0, P1, P2, P3, Alpha) → FVector` | 4-point. Full S-curve control |
| `FastMathBuildBezierArc` | `(P0, P1, P2, P3, NumSteps, OutPoints)` | Fills array for debug draw or spline preview |

**Decisions:**
- `Alpha` is [0, 1] — 0 = P0, 1 = P3
- No catmull-rom or B-spline for now — cubic Bezier covers the stated use cases
- `BuildBezierArc` uses uniform parameterization (not arc-length); enough for preview use

---

### 15. GRID DISTANCE — `"Fast Math|Grid Distance"`

Non-Euclidean distance metrics. No square roots.

| Function | Signature | Notes |
|---|---|---|
| `FastMathManhattanDistance` | `(A, B) → float` | `|dx|+|dy|+|dz|`. 3D grid step count |
| `FastMathManhattanDistance2D` | `(A, B) → float` | `|dx|+|dy|`. 2D tile/maze step count |
| `FastMathChebyshevDistance` | `(A, B) → float` | `max(|dx|,|dy|,|dz|)`. King moves on a chessboard |
| `FastMathOctileDistance` | `(A, B) → float` | Diagonal-move corrected 2D: `D*(dx+dy) + (D√2-D)*min(dx,dy)`. Best 2D heuristic for A* with diagonals |

**Decisions:**
- All inputs are FVector for BP compatibility — internally computes per-axis differences
- Octile is not in the original list but rounds out the set — it's the correct heuristic for
  8-directional tile pathfinding and trivially cheap

---

## Proposed Additions

These are not in the original list but fill obvious gaps and fit the plugin's scope.

### A. HASH — `"Fast Math|Hash"`

Deterministic integer hashing for procedural seeding. Replaces `Random Stream` + `Set Seed` patterns.

| Function | Signature | Notes |
|---|---|---|
| `FastMathHashInt` | `(Value) → int32` | Wang hash — deterministic chaos from one integer |
| `FastMathHashInts2D` | `(X, Y, Seed) → int32` | Spatial hash for 2D grid cells |
| `FastMathHashInts3D` | `(X, Y, Z, Seed) → int32` | Same, 3D |
| `FastMathHashToFloat` | `(Hash) → float` | Maps int32 → [0, 1] uniformly |

**Why:** Procedural generation needs the same value at the same location every time. Random Stream
is stateful and order-dependent. These are pure functions with no state.

---

### B. ANGLE — `"Fast Math|Angle"`

Angle arithmetic that Blueprint handles poorly (wrapping, sign, direction).

| Function | Signature | Notes |
|---|---|---|
| `FastMathShortestAngle` | `(FromDeg, ToDeg) → float` | Signed delta always in [-180, 180]. Fixes "spin the wrong way" bugs. |
| `FastMathNormalizeAngle` | `(Deg) → float` | Wraps to [0, 360). Same as WrapFloat(Deg, 0, 360) but clearer intent. |
| `FastMathHeadingFromVector` | `(Direction) → float` | 2D yaw from XY plane projection (atan2). Replaces "RotationFromXVector → Yaw" |

**Why:** `ShortestAngle` in particular is a recurring trap — without it, interpolating from 350° to
10° goes the wrong way. This is trivial in C++ and a node cluster in Blueprint.

---

### C. REMAP — `"Fast Math|Remap"`

Input normalization and dead zone handling.

| Function | Signature | Notes |
|---|---|---|
| `FastMathRemap` | `(Value, InMin, InMax, OutMin, OutMax) → float` | Unclamped linear remap. UE has MapRangeClamped — this skips the clamp for cases where you want extrapolation. |
| `FastMathDeadzone` | `(Input, InnerRadius, OuterRadius) → float` | Remaps input to [0,1] after dead zone. Input < InnerRadius → 0. Input > OuterRadius → 1. |

**Why:** Deadzone is used for every analog stick, every radial menu, every proximity trigger. The
Blueprint version is 6 nodes. This is 1.

---

## Files to Create

```
Plugins/FastMath/
  FastMath.uplugin
  Source/
    FastMath/
      FastMath.Build.cs
      FastMath.h
      FastMath.cpp
      FastMathFunctionLibrary.h      ← all UFUNCTION declarations
      FastMathFunctionLibrary.cpp    ← all implementations
      FastMathTypes.h                ← EFastMathEase enum only
```

No editor module needed — all nodes are runtime-safe pure math.

---

## Implementation Order

- [ ] Plugin scaffold (uplugin, Build.cs, module h/cpp)
- [ ] `FastMathTypes.h` — EFastMathEase enum
- [ ] `FastMathFunctionLibrary.h` — all declarations
- [ ] Bitmask (trivial, validates build pipeline)
- [ ] Grid Distance + Wrap + Angle (pure math, no state)
- [ ] Remap + Hash + Wave (pure math)
- [ ] Ease (enum-switch, self-contained formulas)
- [ ] Noise (permutation table setup)
- [ ] Weighted Random + Morton
- [ ] Bezier + Trajectory
- [ ] Grid Snap (hex requires careful cube-rounding)
- [ ] Spring + Moving Average (stateful — needs caller-owned variables)
- [ ] Spin Deflect + Rotation (vector/quat math)
- [ ] Full review pass: simplify any function that could be simpler, cut any that duplicate UE built-ins
