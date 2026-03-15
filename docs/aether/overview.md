# Aether — Overview

## Architecture

```
UAetherSubsystem  (UEngineSubsystem)
    └── FAetherViewExtension  (FSceneViewExtensionBase)
            └── PrePostProcessPass_RenderThread()
                    ├── StructureTensor.usf    — Compute per-pixel orientation and anisotropy
                    ├── GaussianBlur.usf       — Smooth the structure tensor (2-pass separable)
                    ├── KuwaharaAKF.usf        — Apply anisotropic sector-weighted filter
                    └── Composite.usf          — Blend AKF result with original at blend strength
```

## Filter Theory

### Classical Kuwahara Filter

The original Kuwahara filter divides a square neighbourhood around each pixel into four overlapping quadrants. It selects the quadrant with the lowest variance and uses its mean as the output color. This naturally smooths textures while preserving edges — but produces angular, blocky artifacts aligned to the image axes.

### Anisotropic Extension

Aether uses the Generalized Anisotropic Kuwahara Filter (Kyprianidis et al.). Instead of four axis-aligned sectors, the filter uses a set of rotated, elongated elliptical sectors aligned to the **local structure tensor** of the image:

1. **Structure Tensor** — Computes the gradient covariance at each pixel, giving a 2×2 symmetric matrix that encodes local edge orientation and contrast.
2. **Tensor Smoothing** — The structure tensor is Gaussian-blurred to reduce noise sensitivity and to spread orientation information over a neighbourhood.
3. **Anisotropic Sectors** — Eight elliptical sectors are rotated to align with the dominant gradient direction. Elongation ratio is proportional to the tensor's eigenvalue ratio (how directional the local structure is).
4. **Weighted Mean Selection** — Each sector computes a weighted mean and variance. The output is the weighted average of all sector means, with weights derived from their variances (low variance → high weight).

The result is smooth, paint-like strokes that follow the natural flow of shapes in the image.

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Kernel radius** | Controls the size of the paint strokes. Larger = coarser, more abstract. |
| **Sharpness (alpha)** | Weighting exponent. Higher = more selective sector weighting, sharper edges. |
| **Orientation sensitivity** | How strongly the kernel elongates along structure tensor eigenvectors |
| **Blend strength** | Linear blend between AKF output and original scene color. 0 = original, 1 = full painterly. |
| **Half-resolution option** | AKF can run at 50% res for performance via `r.Aether.ResScale 0.5` |
