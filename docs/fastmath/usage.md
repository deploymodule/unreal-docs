# FastMath — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"FastMath"` to your `Build.cs`.

## 2. Interpolation

```cpp
// Spring interpolation (dt-stable, no overshooting)
float CurrentFOV = UFastMathBFL::SpringInterp(CurrentFOV, TargetFOV, SpringState, DeltaTime, 10.f, 0.8f);

// Cubic Bezier (UI animation)
FVector P = UFastMathBFL::BezierPoint(P0, P1, P2, P3, T);

// SmoothStep (0→1 ease)
float Alpha = UFastMathBFL::SmoothStep(0.f, 1.f, RawAlpha);
```

## 3. Procedural Noise

```cpp
// FBM for terrain height variation
float Height = UFastMathBFL::FBM(
    FVector2D(X * 0.01f, Y * 0.01f),
    6,      // octaves
    2.0f,   // lacunarity
    0.5f    // gain
);

// Voronoi for cell-based patterns
FFastMathVoronoiResult Vor = UFastMathBFL::VoronoiNoise2D(UV);
float CellEdge = Vor.F2 - Vor.F1;  // edge detection
```

## 4. Spatial Geometry

```cpp
// Closest point on a line segment (for projectile prediction)
FVector Closest = UFastMathBFL::ClosestPointOnSegment(ProjectilePos, LineStart, LineEnd);

// Ray vs sphere (custom hit test)
FFastMathRayHit Hit;
bool bHit = UFastMathBFL::RayVsSphere(RayOrigin, RayDir, SphereCenter, SphereRadius, Hit);

// Angle between vectors (signed, in degrees)
float Angle = UFastMathBFL::SignedAngleBetweenVectors(V1, V2, UpVector);
```

## 5. Quaternion Utilities

```cpp
// Decompose rotation into swing (XY) and twist (Z)
FQuat Swing, Twist;
UFastMathBFL::QuatSwingTwist(MyQuat, FVector::UpVector, Swing, Twist);

// Ensure slerp takes short arc
FQuat Target = UFastMathBFL::QuatShortestPath(Current, DesiredQuat);
```

## 6. Remap

```cpp
// Map damage (0–100) to screen shake intensity (0.0–2.0)
float ShakeIntensity = UFastMathBFL::RemapClamped(Damage, 0.f, 100.f, 0.f, 2.f);
```

## 7. Bit Math

```cpp
// Morton code for spatial hash
uint32 MortonCode = UFastMathBFL::InterleaveBits(GridX, GridY);
int32 NextPow2    = UFastMathBFL::NextPowerOfTwo(Count);
```
