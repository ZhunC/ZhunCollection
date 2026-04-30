---
tags:
  - linear_algebra
---
# PCA (Principal Component Analysis) — quick summary

## What PCA is
PCA finds a new set of **orthogonal axes** (directions) such that:
- **PC1** captures the **maximum variance** (spread) of the data,
- **PC2** captures the next most variance **subject to being perpendicular to PC1**,
- and so on.

You can think of it as **rotating the coordinate system** to align with the data’s main “elongation” directions.

---

## For a 2D scatter: what you get
Given a cloud of points in 2D:
- **PC1 = the principal direction** along which the points vary the most (the “long axis” of the cloud).
- **PC2 = the perpendicular direction** capturing leftover variation.

The “PCA line” in 2D is:
- a line through the **mean** of the data,
- oriented along **PC1**.

---

## Is PCA a linear fit?
Sort of, but **not the same as standard regression**.

### PCA (best-fit line meaning)
- Fits a line that **minimizes squared *perpendicular* distances** from points to the line  
  (a.k.a. orthogonal regression / total least squares geometry).
- Treats x and y **symmetrically** (no “input vs output”).

### Ordinary least squares regression (y = ax + b)
- Minimizes squared **vertical** errors (in y only).
- Assumes x is the predictor and y is the dependent variable.

**When do they look similar?**
- When the cloud is strongly elongated (high correlation) and noise is fairly balanced.

---

## How PCA is computed (conceptual steps)
1. **Center** the data: subtract the mean.
2. Compute the **covariance matrix** of the centered data.
3. Take the **eigenvectors/eigenvalues** of the covariance matrix:
   - eigenvector with largest eigenvalue → **PC1**
   - next eigenvector → **PC2**, etc.

---

## What PCA is good for
- Finding dominant directions of variation (principal axes).
- Dimensionality reduction (keep top k PCs).
- Compression / denoising (often, if noise is isotropic).
- Visualizing structure by projecting onto first 2–3 PCs.

---

## Common gotchas
- **Scaling matters**: if features have different units/magnitudes, PCA will be dominated by the largest scale.
  - Consider **standardization (z-score)** when units differ.
- **Nonlinear shapes**: if data is curved (“banana”), PCA still gives a **straight** direction (may miss structure).
- **Multiple clusters**: PC1 may point between clusters rather than represent either cluster’s internal direction.
- Always **center** before PCA (standard PCA assumes mean-centered data).

---

## One-sentence intuition
PCA finds the direction where projecting the data produces the **largest spread**, then repeats in perpendicular directions.
