# FD-Net with Auxiliary Time Steps: Fast Prediction of PDEs using Hessian-Free Trust-Region Methods

**Authors:** Murat Gulgec, Aishwarya Kumar, Omer T. Inan, Justin Romberg  
**arXiv:** [https://arxiv.org/abs/1910.12680](https://arxiv.org/abs/1910.12680)

---

##  Overview

This paper proposes **FD-Net**, a data-driven framework for predicting the evolution of partial differential equations (PDEs) directly from spatio-temporal data.  
FD-Net combines **finite-difference interpretability** with **deep learning flexibility**, and introduces:

- **Auxiliary (sub-)time steps** for better numerical stability  
- A **Hessian-free Trust-Region Conjugate Gradient (TRCG)** optimizer for faster, more stable training than Adam  

---

##  Method Summary

### Finite-Difference-Inspired Network
- Convolutional filters mimic **finite-difference stencils** (e.g., 3-point filters for ∂²u/∂x²).
- Layers are linear (no activations/bias) → interpretable as numerical operators.
- Multiple derivative layers can be stacked for higher-order effects.

### Auxiliary Time Steps
- Each global time step Δt is subdivided into *k* smaller “auxiliary” steps.  
- Improves temporal resolution and prediction stability.  
- Each auxiliary step predicts the next intermediate state using learned finite-difference operations.

### Optimization
- Compared **Adam** vs **Hessian-free Trust-Region Conjugate Gradient (TRCG)**.  
- TRCG approximates curvature via **Hessian-vector products**, avoiding explicit second-order matrix storage.  
- Better handling of saddle points and large curvature variations.

---

## Experiments

- **System:** 1D Heat Equation (∂u/∂t = β ∂²u/∂x²)  
- **Dataset:** 200 synthetic trajectories, \( x \in [0, \pi] \), \( T = 1000 \), 75/25 train/test  
- **Evaluation:** Stable vs. unstable Δt regimes  

### Key Results
| Condition | Optimizer | Error | Stability |
|------------|------------|--------|------------|
| Stable regime | Adam | ~1e-2 | ✓ |
| Stable regime | TRCG | ~1e-5 | ✓ |
| Unstable regime | Adam | Divergent | ✗ |
| Unstable regime | TRCG | Stable | ✓ |

- Increasing auxiliary steps *k* → smoother, more accurate temporal evolution.  
- FD-Net + TRCG accurately predicts long-term dynamics where explicit Euler fails.

---

## Contributions

1. **Interpretable neural discretization:** Learnable FD-like stencils within a neural framework.  
2. **Auxiliary time-step integration:** A simple yet effective scheme for stability in long-term PDE prediction.  
3. **Second-order training:** Demonstrated significant improvement using TRCG over first-order methods (Adam).  
4. **Robust to unstable Δt:** Handles regimes where explicit solvers fail.

---

## Limitations & Future Work

- Evaluated only on simple 1D linear PDEs (heat equation).  
- Scalability to nonlinear or higher-dimensional PDEs (e.g., Navier–Stokes) not tested.  
- TRCG adds computational complexity for large networks.  
- Non-local PDEs or irregular geometries may require architectural changes.

---

## Takeaway

FD-Net bridges **numerical analysis** and **neural networks**, showing that:
> “Finite-difference-like architectures + second-order optimization = data-efficient, stable PDE prediction.”

This work anticipates modern physics-informed networks that embed structure and numerical stability into learnable operators.

---
