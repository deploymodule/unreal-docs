# FastMath — API Reference

## UFastMathBFL

`#include "FastMathBFL.h"` — All functions are static Blueprint nodes.

### Interpolation

| Function | Signature | Description |
|----------|-----------|-------------|
| `SmoothStep` | `(Min, Max, X) → float` | Hermite ease in/out |
| `SmootherStep` | `(Min, Max, X) → float` | C2 smooth step |
| `SpringInterp` | `(Current, Target, State, DT, Stiffness, Damping) → float` | Spring interpolation |
| `BezierPoint` | `(P0, P1, P2, P3, T) → FVector` | Cubic Bezier evaluation |
| `CatmullRomPoint` | `(P0, P1, P2, P3, T) → FVector` | Catmull-Rom spline |
| `EaseInBounce` | `(T) → float` | Bounce ease in |
| `EaseOutElastic` | `(T, Amplitude, Period) → float` | Elastic ease out |

### Noise

| Function | Signature | Description |
|----------|-----------|-------------|
| `ValueNoise2D` | `(UV) → float` | 2D value noise (0–1) |
| `PerlinNoise2D` | `(UV) → float` | 2D Perlin noise (-1 to 1) |
| `SimplexNoise2D` | `(UV) → float` | 2D simplex noise (-1 to 1) |
| `PerlinNoise3D` | `(XYZ) → float` | 3D Perlin noise |
| `FBM` | `(UV, Octaves, Lacunarity, Gain) → float` | Fractal Brownian motion |
| `VoronoiNoise2D` | `(UV) → FFastMathVoronoiResult` | F1, F2 distances and cell ID |
| `DomainWarpNoise` | `(UV, WarpStrength) → float` | Domain-warped FBM |

### Geometry

| Function | Signature | Description |
|----------|-----------|-------------|
| `ClosestPointOnSegment` | `(Point, SegA, SegB) → FVector` | Nearest point on segment |
| `ClosestPointOnTriangle` | `(Point, A, B, C) → FVector` | Nearest point on triangle |
| `PointInTriangle` | `(Point, A, B, C) → bool` | Barycentric inclusion test |
| `RayVsSphere` | `(Origin, Dir, Center, Radius, Hit) → bool` | Ray-sphere intersection |
| `RayVsPlane` | `(Origin, Dir, PlanePoint, PlaneNormal, Hit) → bool` | Ray-plane intersection |
| `AngleBetween` | `(A, B) → float` | Unsigned angle in degrees |
| `SignedAngleBetweenVectors` | `(A, B, Up) → float` | Signed angle in degrees |
| `VectorProjectOntoPlane` | `(V, Normal) → FVector` | Project vector onto plane |

### Quaternion

| Function | Signature | Description |
|----------|-----------|-------------|
| `QuatShortestPath` | `(From, To) → FQuat` | Ensure short-arc slerp |
| `QuatSwingTwist` | `(Q, TwistAxis, Swing, Twist) → void` | Decompose rotation |
| `QuatLookAt` | `(Forward, Up) → FQuat` | Rotation from direction |
| `QuatDelta` | `(From, To) → FQuat` | Rotation delta |

### Statistical / Range

| Function | Signature | Description |
|----------|-----------|-------------|
| `Remap` | `(Value, InMin, InMax, OutMin, OutMax) → float` | Range mapping |
| `RemapClamped` | `(Value, InMin, InMax, OutMin, OutMax) → float` | Clamped range mapping |
| `WeightedAverage` | `(Values, Weights) → float` | Weighted mean |
| `StandardDeviation` | `(Values) → float` | Population std dev |

### Bit Math

| Function | Signature | Description |
|----------|-----------|-------------|
| `NextPowerOfTwo` | `(Value) → int32` | Next power of 2 |
| `IsPowerOfTwo` | `(Value) → bool` | Power of 2 check |
| `InterleaveBits` | `(X, Y) → uint32` | Morton Z-order encoding |
