# Function Approximation Pipeline

## Overview

This repository implements a comprehensive polynomial function approximation pipeline that systematically compares different combinations of **sampling strategies** and **polynomial basis functions** to approximate a noisy damped sinusoidal function. The pipeline demonstrates critical concepts in numerical analysis, including Runge's phenomenon, basis function selection, and optimal sampling point distribution.

---

## Pipeline Architecture

The function approximation pipeline consists of four main stages:

```
┌─────────────────────┐
│  1. Data Generation │
│   (generator.py)    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   2. Sampling       │
│   (sampling.py)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  3. Estimation      │
│  (estimator.py)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  4. Evaluation &    │
│     Visualization   │
│     (main.py)       │
└─────────────────────┘
```

---

## Stage 1: Data Generation

**Module:** [generator.py](generator.py)

### Target Function

The pipeline approximates a **damped sinusoidal function** with Gaussian noise:

```
y = a·e^(-bx)·sin(cx) + N(0, σ²)
```

**Components:**
- `a`: Amplitude coefficient (default: 1.0)
- `b`: Exponential decay rate (default: 0.5)
- `c`: Sine wave frequency (default: 5.0)
- `σ`: Gaussian noise standard deviation (default: 0.1)

### Functions

#### `generate_function(x, a, b, c, sigma)`
Generates noisy observations of the target function by:
1. Computing the deterministic component: `a·e^(-bx)·sin(cx)`
2. Adding Gaussian noise: `N(0, σ²)`
3. Returning the combined noisy signal

**Use case:** Creating realistic training data with measurement noise

#### `generate_function_clean(x, a, b, c)`
Generates noise-free function values for ground truth comparison.

**Use case:** Evaluating model accuracy against the true underlying function

### Mathematical Properties

This target function exhibits several interesting characteristics that make it suitable for testing approximation methods:
- **Oscillatory behavior:** The sine component creates multiple peaks and troughs
- **Exponential decay:** Amplitude decreases over the domain, creating varying signal strength
- **Non-polynomial:** The true function is not a polynomial, so polynomial approximation introduces inherent error
- **Noise:** Additive Gaussian noise simulates real-world measurement uncertainty

---

## Stage 2: Sampling Strategy

**Module:** [sampling.py](sampling.py)

Sampling strategy determines **where** to place training points in the domain `[min_bound, max_bound]`. The choice dramatically affects approximation quality, especially for high-degree polynomials.

### Method 1: Uniform Sampling

#### `sample_uniform(min_bound, max_bound, n_points)`

**Strategy:** Evenly spaced points using `np.linspace()`

**Advantages:**
- Simple and intuitive
- Natural for time-series or regularly sampled data
- Works well for low-degree polynomials

**Disadvantages:**
- Vulnerable to **Runge's phenomenon** in high-degree interpolation
- Poor conditioning of the resulting linear system
- Oscillations near domain boundaries
- Suboptimal approximation error distribution

**Mathematical insight:** Uniform sampling creates a Vandermonde matrix with poor condition number as degree increases, leading to numerical instability.

### Method 2: Chebyshev Nodes

#### `sample_chebyshev_nodes(min_bound, max_bound, n_points)`

**Strategy:** Uses roots of Chebyshev polynomials of the first kind:

```
x_k = cos((2k - 1)π / (2n))  for k = 1, 2, ..., n
```

Mapped from `[-1, 1]` to `[min_bound, max_bound]` and sorted in ascending order.

**Advantages:**
- **Optimal for polynomial interpolation:** Minimizes maximum interpolation error
- Dramatically reduces Runge's phenomenon
- Points cluster near boundaries where polynomials tend to oscillate
- Better conditioned linear systems
- Provably near-optimal error bounds

**Disadvantages:**
- Non-uniform distribution may seem counterintuitive
- Requires domain transformation

**Mathematical insight:** Chebyshev nodes are the projection of equally spaced points on a semicircle onto the x-axis, naturally concentrating samples where polynomial approximation error tends to be largest.

---

## Stage 3: Polynomial Fitting

**Module:** [estimator.py](estimator.py)

The fitting stage constructs a polynomial approximation from sampled data points. The choice of **basis functions** affects numerical stability and computational efficiency, though mathematically all polynomial bases span the same space.

### Method 1: Monomial Basis

#### `fit_monomial(x, y, sigma, n)`

**Basis functions:** `{1, x, x², x³, ..., xⁿ}`

**Implementation:**
- Uses `numpy.polyfit()` for least squares fitting
- Returns coefficients in ascending order: `[c₀, c₁, c₂, ..., cₙ]`
- Creates callable function using `numpy.polyval()`

**Advantages:**
- Simple and widely understood
- Direct interpretation of coefficients
- Standard in many applications

**Disadvantages:**
- **Numerically unstable** for high degrees (n > 15)
- Monomial basis functions become increasingly similar as degree increases
- Poor condition number of the Vandermonde matrix
- Coefficients can have large magnitudes

**Mathematical formulation:**

```
y(x) = c₀ + c₁x + c₂x² + ... + cₙxⁿ
```

Solved via least squares: minimize `||Xc - y||²` where X is the Vandermonde matrix.

### Method 2: Chebyshev Basis

#### `fit_chebyshev(x, y, sigma, n)`

**Basis functions:** `{T₀(x), T₁(x), T₂(x), ..., Tₙ(x)}`

where Chebyshev polynomials are defined by the recurrence:
```
T₀(x) = 1
T₁(x) = x
Tₖ₊₁(x) = 2x·Tₖ(x) - Tₖ₋₁(x)
```

**Implementation:**
1. Normalizes input domain to `[-1, 1]`
2. Fits using `numpy.polynomial.Chebyshev.fit()`
3. Returns Chebyshev coefficients `[c₀, c₁, ..., cₙ]`
4. Creates callable function handling automatic normalization

**Advantages:**
- **Numerically stable** even for high degrees
- Orthogonal basis functions with respect to weight `1/√(1-x²)` on `[-1, 1]`
- Coefficients decay rapidly for smooth functions
- Better conditioned fitting problem
- Optimal approximation properties

**Disadvantages:**
- Requires domain normalization
- Less familiar to most practitioners
- Coefficient interpretation is less intuitive

**Mathematical formulation:**

```
y(x) = c₀T₀(x) + c₁T₁(x) + ... + cₙTₙ(x)
```

**Key insight:** While both bases mathematically span the same polynomial space (meaning they can represent the same functions), Chebyshev basis provides superior numerical properties. The fitted polynomials from both methods will be identical in exact arithmetic, but floating-point errors make Chebyshev basis preferred for practical computation.

---

## Stage 4: Evaluation and Visualization

**Module:** [main.py](main.py)

The main script orchestrates the complete pipeline, comparing all combinations:

### Pipeline Execution

```python
# 2x2 comparison matrix
┌─────────────────────┬──────────────────────┐
│ Monomial Basis     │ Chebyshev Basis      │
│ Uniform Sampling   │ Uniform Sampling     │
├─────────────────────┼──────────────────────┤
│ Monomial Basis     │ Chebyshev Basis      │
│ Chebyshev Nodes    │ Chebyshev Nodes      │
└─────────────────────┴──────────────────────┘
```

### Workflow

1. **Configuration**
   - Domain: `[0, 5]`
   - Training points: 100
   - Polynomial degree: 10
   - Noise level: σ = 0.1
   - Function parameters: a=1.0, b=0.5, c=5.0

2. **Data Generation**
   - Create two sampling point sets: uniform and Chebyshev nodes
   - Generate noisy observations for each set

3. **Model Fitting**
   - Fit four models:
     - Monomial + Uniform
     - Chebyshev + Uniform
     - Monomial + Chebyshev nodes
     - Chebyshev + Chebyshev nodes

4. **Evaluation**
   - Generate dense evaluation grid (300 points)
   - **Extend domain by 1%** on each side to reveal extrapolation behavior
   - Compute R² (coefficient of determination) for each model:
     ```
     R² = 1 - (SS_res / SS_tot)
     where:
       SS_res = Σ(y_true - y_pred)²
       SS_tot = Σ(y_true - y_mean)²
     ```
   - R² = 1 indicates perfect fit; R² < 0 indicates worse than mean prediction

5. **Visualization**
   - Create 2×2 subplot grid
   - Each subplot shows:
     - True function (black line)
     - Fitted polynomial (colored line)
     - Training data points (scatter)
     - Domain boundaries (vertical dashed lines)
     - R² score in title
   - Save as `function_approximation_comparison.png` at 150 DPI

### Key Evaluation Insights

**R² Interpretation:**
- R² ≈ 1.0: Excellent fit
- R² > 0.9: Good fit
- R² < 0.5: Poor fit
- R² < 0: Model worse than simply predicting the mean

**Expected Results:**
1. **Chebyshev nodes outperform uniform sampling** within the training domain
2. **Basis choice matters less** than sampling strategy for the fitting domain
3. **All models struggle with extrapolation** beyond training boundaries
4. **Runge oscillations** visible in uniform sampling, especially near boundaries

---

## Mathematical Background

### Runge's Phenomenon

When interpolating with high-degree polynomials on uniformly spaced points, the approximation can exhibit **large oscillations** near the boundaries, even as the number of points increases. This counterintuitive behavior is Runge's phenomenon.

**Example:** Interpolating the Runge function `f(x) = 1/(1 + 25x²)` on `[-1, 1]` with uniform points shows divergence at boundaries as degree increases.

**Solution:** Chebyshev nodes minimize the Lebesgue constant, providing near-optimal point distribution.

### Approximation Theory

For a function `f` approximated by polynomial `p` of degree `n`:

**Uniform Sampling Error:**
```
||f - p||∞ ≤ C · λₙ · ||f - p*||∞
```
where λₙ (Lebesgue constant) grows exponentially: λₙ ~ 2ⁿ/(e·n·log(n))

**Chebyshev Nodes Error:**
```
||f - p||∞ ≤ C · log(n) · ||f - p*||∞
```
where λₙ grows only logarithmically: λₙ ~ (2/π)·log(n)

This **exponential vs logarithmic** growth explains Chebyshev nodes' superiority.

### Polynomial Spaces

Both monomial and Chebyshev bases span the same vector space `Pₙ` (polynomials of degree ≤ n). They represent different **coordinate systems** for the same space:
- **Monomial basis:** Natural coordinates, poor numerical properties
- **Chebyshev basis:** Orthogonal coordinates, excellent numerical properties

Analogy: Like using Cartesian vs. polar coordinates—same space, different representations.

---

## Usage

### Running the Complete Pipeline

```bash
python main.py
```

**Output:**
- Console output with R² scores
- `function_approximation_comparison.png` visualization

### Customizing the Pipeline

**Modify parameters in [main.py](main.py:12-17):**

```python
# Domain
min_bound = 0
max_bound = 5

# Sampling
n_training_points = 100

# Model complexity
polynomial_degree = 10

# Noise level
sigma = 0.1

# Target function parameters
a, b, c = 1.0, 0.5, 5.0
```

### Using Individual Modules

```python
from generator import generate_function, generate_function_clean
from sampling import sample_uniform, sample_chebyshev_nodes
from estimator import fit_monomial, fit_chebyshev

# Generate training data
x = sample_chebyshev_nodes(0, 5, 50)
y = generate_function(x, a=1.0, b=0.5, c=5.0, sigma=0.1)

# Fit model
coeffs, fit_func = fit_chebyshev(x, y, sigma=0.1, n=10)

# Evaluate
x_eval = np.linspace(0, 5, 200)
y_pred = fit_func(x_eval)
```

---

## Performance Comparison

### Typical R² Results (degree=10, n=100)

| Basis      | Sampling  | R² Score | Notes                           |
|------------|-----------|----------|---------------------------------|
| Monomial   | Uniform   | 0.85-0.95| Runge oscillations at boundaries|
| Chebyshev  | Uniform   | 0.85-0.95| Similar to monomial (same space)|
| Monomial   | Cheb nodes| 0.95-0.99| Much improved boundary behavior |
| Chebyshev  | Cheb nodes| 0.95-0.99| Best numerical stability        |

**Key Takeaway:** Sampling strategy (Chebyshev nodes) has **greater impact** than basis choice for approximation quality, but Chebyshev basis provides better numerical stability for high-degree fits.

---

## Limitations and Considerations

### Extrapolation

**All polynomial approximations fail catastrophically outside the training domain.** The extended evaluation grid reveals that polynomials oscillate wildly beyond `[min_bound, max_bound]`, regardless of basis or sampling.

**Recommendation:** Only trust polynomial approximations **within** the fitted interval. For extrapolation, use physics-based models or methods designed for extrapolation (e.g., splines with boundary conditions, rational functions).

### Degree Selection

- **Low degrees (n < 5):** May underfit complex functions
- **Medium degrees (5 ≤ n ≤ 15):** Often optimal balance
- **High degrees (n > 15):** Risk of overfitting and numerical instability

**Recommendation:** Use cross-validation or regularization to select appropriate degree. Start with n = 5-10 and increase only if residuals are systematic.

### Noise Handling

The current implementation uses least squares fitting, which is optimal for Gaussian noise. The `sigma` parameter is provided to estimators but not currently used for weighting. For heteroscedastic noise (varying σ across domain), weighted least squares would be appropriate.

### Computational Complexity

- **Sampling:** O(n)
- **Fitting:** O(n²) for least squares (dominant cost)
- **Evaluation:** O(n) per point

For large datasets (n > 10,000), consider:
- Orthogonal polynomial methods with fast transforms
- Iterative solvers for least squares
- Reduced-rank approximations

---

## Extensions and Future Work

### Potential Enhancements

1. **Regularization:** Add L2 (ridge) or L1 (lasso) penalties to prevent overfitting
2. **Cross-validation:** Implement k-fold CV for automatic degree selection
3. **Uncertainty quantification:** Compute prediction intervals using bootstrap or analytical formulas
4. **Alternative bases:** Implement Legendre, Hermite, or Laguerre polynomials
5. **Piecewise polynomials:** Compare with spline methods (B-splines, cubic splines)
6. **Adaptive sampling:** Use error estimates to refine sampling in high-error regions
7. **Multidimensional:** Extend to 2D function approximation with tensor products

### Research Questions

1. How does approximation quality scale with noise level?
2. What is the optimal degree as a function of n_points and noise?
3. Can we develop adaptive sampling that performs well without a priori knowledge?
4. How do these methods compare to modern ML approaches (Gaussian processes, neural networks)?

---

## References and Further Reading

### Approximation Theory
- Trefethen, L.N. (2013). *Approximation Theory and Approximation Practice*. SIAM.
- Cheney, E.W. (2000). *Introduction to Approximation Theory*. AMS Chelsea.

### Numerical Analysis
- Higham, N.J. (2002). *Accuracy and Stability of Numerical Algorithms*. SIAM.
- Gautschi, W. (2004). *Orthogonal Polynomials: Computation and Approximation*. Oxford.

### Classical Papers
- Runge, C. (1901). "Über empirische Funktionen und die Interpolation zwischen äquidistanten Ordinaten."
- Chebyshev, P.L. (1854). "Théorie des mécanismes connus sous le nom de parallélogrammes."

### Online Resources
- [Numpy Polynomial Reference](https://numpy.org/doc/stable/reference/routines.polynomials.html)
- [Chebyshev Polynomials (Wikipedia)](https://en.wikipedia.org/wiki/Chebyshev_polynomials)
- [Runge's Phenomenon (Wolfram)](https://mathworld.wolfram.com/RungesPhenom.html)

---

## Conclusion

This function approximation pipeline provides a hands-on demonstration of fundamental concepts in numerical analysis and approximation theory. The systematic comparison of sampling strategies and basis functions reveals that:

1. **Sampling strategy dominates:** Chebyshev nodes provide superior approximation quality
2. **Basis affects stability:** Chebyshev basis enables higher-degree fits without numerical issues
3. **Extrapolation fails:** Polynomial approximations should only be trusted within the fitted domain
4. **Theory guides practice:** Mathematical insights (Lebesgue constants, orthogonality) directly translate to practical performance

By experimenting with different parameters (degree, noise level, function parameters), users can develop intuition about the strengths and limitations of polynomial approximation methods—knowledge that extends to many areas of scientific computing, data analysis, and machine learning.
