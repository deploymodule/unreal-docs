# FastMath — Overview

## Categories

### Interpolation
Extended interpolation functions beyond `Lerp` and `FInterpTo`:
- `SmoothStep` — Hermite curve ease in/out (GPU-equivalent)
- `SmootherStep` — Ken Perlin's smoother step (C2 continuity)
- `SpringInterp` — critically damped spring interpolation (dt-stable)
- `Bounce` / `Elastic` — easing functions for UI animation
- `BezierPoint` — evaluate a cubic Bezier at parameter T
- `CatmullRomPoint` — evaluate a Catmull-Rom spline segment

### Procedural Noise
- `ValueNoise2D` / `ValueNoise3D` — lattice value noise
- `PerlinNoise2D` / `PerlinNoise3D` — gradient noise
- `SimplexNoise2D` / `SimplexNoise3D` — simplex noise
- `FBM` — fractal Brownian motion (configurable octaves, lacunarity, gain)
- `VoronoiNoise2D` — cellular/Worley noise, returns F1 and F2 distances
- `DomainWarpNoise` — domain-warped FBM for organic shapes

### Spatial Geometry
- `ClosestPointOnLine` / `ClosestPointOnSegment`
- `ClosestPointOnTriangle`
- `PointInTriangle` — barycentric test
- `RayVsPlane` / `RayVsSphere` / `RayVsBox`
- `SphericalToCartesian` / `CartesianToSpherical`
- `AngleBetweenVectors` (signed and unsigned variants)
- `VectorProjectOntoPlane`

### Quaternion Utilities
- `QuatShortestPath` — ensures slerp takes the short arc
- `QuatSwingTwist` — decompose quaternion into swing and twist components
- `QuatLookAt` — construct a rotation from direction + up vector
- `QuatDelta` — rotation delta between two quaternions

### Statistical Math
- `NormalDistribution` — Gaussian PDF and CDF approximation
- `Remap` — map a value from one range to another
- `RemapClamped` — same with output clamping
- `StandardDeviation` — compute std dev from a float array
- `WeightedAverage` — weighted mean from value+weight arrays

### Bit Math
- `NextPowerOfTwo` / `IsPowerOfTwo`
- `BitCount` / `TrailingZeroes`
- `InterleaveBits` — Morton encoding for spatial hashing
