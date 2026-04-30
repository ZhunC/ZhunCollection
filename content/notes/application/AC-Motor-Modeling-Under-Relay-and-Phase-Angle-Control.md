---

## tags: [motor-control, system-identification, numerical-methods, control-theory] topics: [AC motor modeling, phase-angle control, burst firing, harmonic analysis] related: [[System Identification]], [[Numeric Methods]], [[Control Theory]], [[Fourier Analysis]]
---
# AC Motor Modeling Under Relay and Phase-Angle Control

## Context

Supplier provides a motor model `f(voltage, torque, rpm)` tested with a clean sinusoidal input — only amplitude is varied. Actual usage differs in two ways:

1. **Firing-angle control**: The 60 Hz waveform is phase-chopped per cycle (SCR/TRIAC)
2. **Burst firing**: Only a subset of cycles are active per control period (e.g., 4 of 6 cycles per 0.1s)

The challenge is to either adapt the supplier model or replace it with one that accurately describes the system under these real input conditions.

---

## Problem 2 — Amplitude-Only Control Doesn't Fit

Even before introducing wave chopping, controlling voltage by amplitude alone doesn't reproduce the supplier model. The first step is diagnosing _why_.

### Residual Diagnosis

Compute `r = f_supplier(V, rpm) - T_measured` and characterize it:

|Residual pattern|Interpretation|Action|
|---|---|---|
|Systematic offset or scaling|Parameter drift / calibration mismatch|Re-identify parameters|
|Nonlinear trend with V or rpm|Wrong model structure|New model or correction term|
|Random scatter around zero|Measurement noise only|Denoising / regularization|

> [!important] Don't skip this step. The residual structure determines the correct path forward.

### Option A — Entirely New Empirical Model

Fit a polynomial, GP, or small neural net to `(V, T, rpm)` data directly.

- ✅ Captures actual behavior
- ❌ Poor extrapolation; loses physical interpretability; needs dense data coverage
- **Use only if**: supplier model physics are fundamentally inapplicable to your motor

### Option B — Re-identify Equivalent Circuit Parameters ⭐

The supplier model is almost certainly built on the standard induction motor T-equivalent circuit with parameters `(R₁, L₁, R₂, L₂, Lₘ)`. Their identified values may not match yours due to manufacturing variance, temperature, or instrumentation differences.

Run nonlinear least squares to fit the same model structure to your data:

```python
from scipy.optimize import least_squares

def motor_model(params, V, rpm):
    R1, L1, R2, L2, Lm = params
    # standard slip-torque from equivalent circuit
    ...
    return T_predicted

result = least_squares(
    lambda p: motor_model(p, V_data, rpm_data) - T_data,
    x0=supplier_params,
    bounds=(lower, upper)  # constrain to physically plausible range
)
```

- ✅ Physically interpretable; good extrapolation; fewer degrees of freedom
- **This should be the first attempt**

### Option C — Supplier Model + Residual Correction Term

After re-identification, if structured residual remains:

```
T_actual(V, rpm) = f_supplier(V, rpm) + δ(V, rpm)
```

Fit `δ` as a low-order polynomial or RBF over the residuals.

- ✅ Leverages physical prior; interpretable deviations
- ❌ If `δ` is large, it signals the base model is wrong — don't use as a crutch

### Decision Sequence

```
Residual structured?
   ├─ Yes → Re-identify parameters (Option B)
   │         └─ Still structured? → Add δ correction (Option C)
   └─ No  → Model is fine; denoise measurements
              └─ Massive unexplainable mismatch? → New model (Option A)
```

---

## Problem 1 — Phase-Angle (Firing-Angle) Control

### Why Amplitude ≠ Firing Angle

A phase-angle controller (SCR/TRIAC) with firing angle `α` doesn't simply attenuate voltage. It **truncates the leading edge of each half-cycle**, fundamentally changing the waveform's harmonic content.

The RMS voltage is:

$$V_{\text{rms}}(\alpha) = V_{\text{peak}} \sqrt{\frac{1}{2\pi}\left(\pi - \alpha + \frac{\sin 2\alpha}{2}\right)}$$

But the **waveform shape changes with** `α`, injecting strong odd harmonics (3rd, 5th, 7th, ...) that are absent in the supplier's test conditions.

### Harmonic Decomposition Approach

Each harmonic at frequency `n·ω` contributes its own slip and torque:

$$s_n = \frac{n\omega_s \mp \omega_r}{n\omega_s}$$

Total electromagnetic torque:

$$T_{\text{total}} = T_1(\alpha) + \sum_{n \in {5, 7, 11, \ldots}} T_n(\alpha)$$

> [!note] In practice, only the 5th and 7th harmonics matter. Higher harmonics are attenuated by the motor's leakage inductance (which acts as a low-pass filter).

### Practical Model Extension

Rather than solving the full harmonic sum analytically, re-parameterize and fit empirically:

1. Compute `V_fund(α)` — RMS of the **fundamental Fourier component** (not just naive RMS)
2. Apply the re-identified supplier model to `V_fund`
3. Add a data-fit harmonic derating term `k(α)`:

```python
def derated_model(V_peak, alpha, rpm, params):
    V_fund = compute_fundamental_rms(V_peak, alpha)
    T_base = motor_model(V_fund, rpm, params)
    harmonic_loss = k_alpha(alpha) * T_base
    return T_base - harmonic_loss
```

`k(α)` properties:

- Near zero at `α = 0` (clean waveform)
- Monotonically increasing toward `α = π`
- Fit from your dataset

---

## Problem 1 Extended — Burst Firing + Firing Angle

### The Two-Layer Control Scheme

At 60 Hz, a 0.1s window = **6 cycles**. Combining burst firing and firing angle gives two independent modulation layers:

|Layer|Mechanism|Example|
|---|---|---|
|**Burst firing**|Select which cycles are "on"|4 of 6 active (d = 2/3)|
|**Phase-angle**|Chop leading edge of each active cycle|α = 25°|

### Layer 1 — Burst Firing Dynamics

The motor's mechanical time constant:

$$\tau_m = \frac{J \cdot \omega}{T_{load}}$$

This determines whether the motor averages over the burst or physically responds to it:

**Case A — Large inertia (τ_m >> 0.1s)**

Motor mechanically low-pass filters the burst. Average torque is simply duty-cycle-scaled:

$$T_{avg} = d \cdot T_{\text{model}}(V_{\text{rms}}(\alpha),\ \text{rpm})$$

where `d = N_on / N_total`. Clean and tractable — static model applies.

**Case B — Light load / small inertia (τ_m ~ 0.1s)**

Motor actually accelerates during active cycles and decelerates during dead cycles. Speed ripple at 10 Hz emerges and the static model breaks down. Requires dynamic simulation:

```python
for t in burst_timeline:
    if cycle_is_active(t):
        T_em = motor_model(V_rms(alpha), rpm)
    else:
        T_em = 0
    rpm += (dt / J) * (T_em - T_load(rpm))
```

> [!warning] If measurements are taken at random burst phases in Case B, residuals will appear noisy but are actually structured oscillation. Average over a full burst period before fitting.

### Layer 2 — Firing Angle Within Active Cycles

Same harmonic analysis as Problem 1 above. At α = 25°, distortion is modest. Becomes severe past α ~ 90°.

### Collapsed Effective Input

For quasi-static regime (Case A), the two layers compose:

$$V_{fund,eff} = d \cdot V_{fund}(V_{peak},\ \alpha)$$

Full input space for your model:

```
(V_peak, d, α, rpm)  →  collapse to  →  (V_fund_eff, rpm)
```

### Flux Decay and Cycle Restart Transients

Rotor time constant: `τ_flux = Lₘ / R₂` (typically 50–300 ms)

When active cycles resume after a dead window:

- **Dead window < τ_flux**: Flux still substantial — smooth restart
- **Dead window > τ_flux**: Flux has decayed — mini-inrush at each burst restart

At 4/6 active, dead window = 2 cycles = 33 ms at 60 Hz. This is likely **shorter than τ_flux** for most motors, so transients are mild. Relevant mainly if you observe unexpected current spikes.

---

## Full Modeling Recommendation

```python
# Feature engineering from raw controls
d       = N_on / N_total                          # burst duty cycle
V_fund  = compute_fundamental_rms(V_peak, alpha)  # fundamental RMS from Fourier
V_eff   = d * V_fund                              # collapsed effective voltage

# Model
T_pred = motor_model(V_eff, rpm, params)          # re-identified supplier model
       + k_alpha(alpha) * motor_model(...)         # harmonic derating term
```

### Summary Table

|Problem|Root Cause|Recommended Approach|
|---|---|---|
|Amplitude mismatch (P2)|Parameter mismatch between supplier and your motor|Re-identify equivalent circuit params via NLS|
|Structured residual after NLS|Unmodeled secondary effects|Add residual correction term `δ(V, rpm)`|
|Firing-angle control (P1)|Waveform harmonics not in supplier model|Fourier decompose; apply model to fundamental; fit `k(α)`|
|Burst firing, large inertia|Duty-cycle modulation of torque|Scale voltage input by `d` before applying model|
|Burst firing, small inertia|Speed ripple invalidates static model|Dynamic simulation over burst period|

---

## Key Principles

> Physics first — fit only what you cannot derive. A fully empirical model is the last resort, not the first tool.

- The supplier model encodes physical structure (equivalent circuit). Preserve it; only re-identify its parameters.
- Firing-angle control is a **waveform transformation problem**, not a simple voltage reduction. Fourier decomposition is the right frame.
- Burst firing is a **timescale separation problem**. The answer depends on where your mechanical time constant sits relative to the burst period.
- The effective input `V_fund_eff = d · V_fund(V_peak, α)` collapses both control layers into a single physically meaningful quantity for the quasi-static case.

---

## Connections

- [[Fourier Analysis]] — fundamental RMS extraction, harmonic content of phase-chopped waveforms
- [[System Identification]] — nonlinear least squares for equivalent circuit parameter fitting
- [[Numeric Methods]] — `scipy.optimize.least_squares`, bounded parameter estimation
- [[Control Theory]] — mechanical time constant, slip-torque relationships, flux dynamics