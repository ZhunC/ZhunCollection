# Logistic Regression vs Recursive Least Squares (RLS) for Online Freeze-Point Detection

## 1. Problem Framing

The core problem discussed is detecting a **phase transition (freezing
point)** from a noisy temperature time series in real time.

This is fundamentally a: - **Change-point detection problem** - **Online
system identification problem** - Not merely a static classification
problem

The key challenge: - The freezing point does not always produce a sharp
derivative change. - Supercooling can violate monotonic assumptions. -
The system must operate online with minimal compute and memory. -
Detection must be robust to noise and mixture variability.

------------------------------------------------------------------------

## 2. Logistic Regression Approach

### 2.1 What Logistic Regression Requires

Logistic regression operates on fixed-dimensional feature vectors. To
apply it to time-series data, we must:

1.  **Encode the time series**
    -   Sliding window extraction
    -   Feature engineering (slope, curvature, residuals, variance,
        etc.)
    -   Or learned encoders (CNN/RNN/TCN)
2.  **Define Labels**
    -   Binary labels per window (freeze event vs no freeze)
    -   Or probabilistic tags
    -   Requires offline annotation or synthetic labeling
3.  **Train Offline**
    -   Learn decision boundary: P(freeze \| features)
4.  **Deploy Two Components**
    -   Time-series encoder
    -   Logistic regression inference model
    -   Plus post-processing to extract the final freeze-point time

------------------------------------------------------------------------

### 2.2 Structural Limitations

Logistic regression:

-   Assumes independent samples
-   Does not inherently model temporal causality
-   Does not detect change-points natively
-   Requires manual feature design to capture phase-transition behavior

Effectively, the *hard temporal reasoning* must be built **before**
logistic regression sees the data.

This makes logistic regression a downstream classifier rather than a
true temporal model.

------------------------------------------------------------------------

## 3. Recursive Least Squares (RLS) Approach

### 3.1 Core Idea

RLS is an **online adaptive parameter estimation method**.

It fits a linear model to the cooling curve:

    T(t) ≈ m·t + c

The model parameters are updated recursively:

    θ(t+1) = θ(t) + K(t)·e(t)

Where: - θ = \[m, c\] - e(t) = prediction error - K(t) = adaptive gain -
λ = forgetting factor

------------------------------------------------------------------------

### 3.2 Why It Works for Freeze Detection

Freezing is detected when:

-   The observed temperature deviates significantly from the predicted
    linear trend.
-   This large residual signals a **regime change**.

Thus, freezing detection becomes:

> Detect when the learned cooling model fails.

This is equivalent to residual-based change-point detection.

------------------------------------------------------------------------

### 3.3 Advantages of RLS

-   Fully online learning
-   O(1) state memory
-   Adaptive to changing cooling dynamics
-   Physically interpretable (slope, intercept)
-   Naturally suited for embedded deployment
-   Detects change through model mismatch

------------------------------------------------------------------------

## 4. Conceptual Comparison

  -------------------------------------------------------------------------------
  Aspect             Logistic Regression                         RLS
  ------------------ ------------------------------------------- ----------------
  Problem Framing    Classification                              Online system
                                                                 identification

  Temporal Modeling  Indirect (via features)                     Direct

  Online Learning    Typically no                                Yes

  Change-Point       Requires extra logic                        Emerges via
  Detection                                                      residuals

  Interpretability   Moderate                                    High

  Embedded           Moderate (encoder cost)                     High
  Suitability                                                    

  Physical Grounding Weak                                        Strong
  -------------------------------------------------------------------------------

------------------------------------------------------------------------

## 5. Key Insight

Logistic regression can only work **after** the time series is converted
into structured tokens and labeled appropriately.

RLS, in contrast, models the physical cooling process directly and
detects freezing as a **breakdown of the learned model**.

Therefore:

-   Logistic regression = classifier on engineered temporal summaries\
-   RLS = adaptive model + residual-based change detection

These represent fundamentally different approaches: - One reframes the
problem as classification. - The other treats it as adaptive system
modeling.

------------------------------------------------------------------------

## 6. Alternative Methods Worth Considering

If seeking alternatives better aligned with the temporal nature of the
problem:

-   CUSUM change-point detection
-   Bayesian Online Change Point Detection (BOCPD)
-   Hidden Markov Models (regime switching)
-   Kalman filter + innovation thresholding
-   Piecewise linear segmentation

These preserve the structure of a time-dependent regime change more
naturally than static classification.

------------------------------------------------------------------------

## 7. Final Takeaway

The freeze detection problem is fundamentally:

> An online change-point detection problem under noisy nonlinear
> dynamics.

RLS aligns directly with this structure by: - Learning the system
trend - Detecting deviations - Operating in real time

Logistic regression requires substantial preprocessing and restructuring
of the problem before it becomes applicable.
