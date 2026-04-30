---
title: Whitening in Nonlinear Least-Squares
tags: [estimation, least-squares, optimization, state-estimation]
created: 2026-04-29
---

# Whitening in Nonlinear Least-Squares

## The Core Idea

When measurements have different units or noise levels, you cannot simply stack residuals into one cost — a 1 mm position error and a 1 rad angle error are incommensurable. The fix is **whitening**: pre-multiply each residual block by $\Sigma_i^{-1/2}$, which decorrelates and normalizes it.

This transforms the Mahalanobis-norm cost

$$\|e\|^2_\Sigma = e^\top \Sigma^{-1} e$$

into a standard Euclidean-norm cost on whitened quantities:

$$A_i = \Sigma_i^{-1/2} H_i, \qquad b_i = \Sigma_i^{-1/2}\bigl(z_i - h_i(X_i^0)\bigr)$$

and the problem becomes the familiar $\arg\min_\Delta \|A\Delta - b\|_2^2$.

## First-Order Accuracy Caveat

The Jacobian $H_i$ comes from a **first-order Taylor expansion** of the (possibly nonlinear) measurement function $h_i$:

$$h_i(X_i^0 + \Delta_i) \approx h_i(X_i^0) + H_i \Delta_i$$

This means the whitened least-squares system is only **locally valid** near the linearization point $X_i^0$. The solution $\Delta^*$ should be treated as a correction step, not a global answer — hence iterative solvers (Gauss-Newton, Levenberg-Marquardt) re-linearize and re-whiten at each step.

> **Caution**: if the true $h_i$ is highly nonlinear, the first-order approximation degrades quickly away from $X_i^0$, and a single-step solve can be badly wrong.

## Tips

- **Iterate**: always use the whitened solve as one step inside an outer loop that updates $X^0$ and re-computes $H_i$ and $b_i$.
- **Check covariance consistency**: $\Sigma_i$ must match the actual sensor noise. Underestimating variance makes the optimizer overtrust a noisy sensor; overestimating makes it ignore useful data.
- **Condition number**: $\Sigma_i^{-1/2}$ amplifies small eigenvalues of $\Sigma_i$. If a noise covariance is near-singular (e.g., a nearly deterministic constraint), the whitened system becomes ill-conditioned — add a small regularizer or treat it as a hard constraint instead.
- **Manifold states**: if any component of $X$ lives on a manifold (SO(3), SE(3)), the additive $\Delta$ model breaks down. Use retraction maps (e.g., $\exp$-map) and compute $H_i$ in the tangent space.

## Pitfalls

- **Stale Jacobians**: reusing $H_i$ from a far-off linearization point without re-linearizing leads to slow or divergent convergence.
- **Unit mismatch in $\Sigma$**: mixing units inside $\Sigma_i$ (e.g., meters² and radians² on the diagonal) produces a numerically correct but physically meaningless whitening if the cross-terms are wrong.
- **Outliers**: whitening assumes Gaussian noise. A single large outlier inflates the cost and can dominate the solution. Use robust norms (Huber, Cauchy) or RANSAC-style rejection before whitening.
- **Over-parameterization**: if $A$ is rank-deficient after stacking all factors, the system is underdetermined. Add a prior (regularization) or fix gauge freedoms before solving.
