# FastMath — Changelog

## v1.0.0 — Initial Release

### Added
- `UFastMathBFL` — Blueprint Function Library with 40+ native math functions
- **Interpolation**: SmoothStep, SmootherStep, SpringInterp (dt-stable), BezierPoint, CatmullRomPoint, EaseInBounce, EaseOutElastic
- **Noise**: ValueNoise2D/3D, PerlinNoise2D/3D, SimplexNoise2D/3D, FBM (configurable octaves/lacunarity/gain), VoronoiNoise2D, DomainWarpNoise
- **Geometry**: ClosestPointOnSegment, ClosestPointOnTriangle, PointInTriangle, RayVsSphere, RayVsPlane, AngleBetween, SignedAngleBetweenVectors, VectorProjectOntoPlane
- **Quaternion**: QuatShortestPath, QuatSwingTwist, QuatLookAt, QuatDelta
- **Statistical**: Remap, RemapClamped, WeightedAverage, StandardDeviation
- **Bit math**: NextPowerOfTwo, IsPowerOfTwo, InterleaveBits (Morton encoding)
- `FFastMathVoronoiResult` struct exposing F1, F2, and cell ID from Voronoi query
- `FFastMathRayHit` struct exposing hit point, normal, and T distance
- `FFastMathSpringState` struct for persistent spring interpolation state
- SIMD paths for noise and geometry where available
- All functions Blueprint-callable under the **FastMath** category
