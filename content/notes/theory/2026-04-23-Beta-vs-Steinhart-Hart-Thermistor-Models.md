---
title: Beta vs Steinhart-Hart Equations for Thermistor Modeling
tags: [thermistor, temperature-sensing, NTC, resistance-temperature, modeling]
created: 2026-04-23
---

## The Beta (β) equation
A **simplified two-parameter model** for NTC (Negative Temperature Coefficient) thermistors:

$$\frac{1}{T} = \frac{1}{T_0} + \frac{1}{\beta}\ln\left(\frac{R}{R_0}\right)$$

Where:
- $T$ = absolute temperature (K)
- $R$ = resistance at temperature $T$ (Ω)
- $T_0$ = reference temperature (typically 298.15 K / 25°C)
- $R_0$ = resistance at $T_0$ (Ω)
- $\beta$ = material constant (K), typically 3000–4000 K

**Characteristics:**
- Simple, only **2 parameters** ($R_0$, $\beta$)
- Accurate over **narrow temperature ranges** (~50°C span)
- Error increases at temperature extremes

---

## The Steinhart-Hart equation
A **three-parameter model** with higher accuracy:

$$\frac{1}{T} = A + B\ln(R) + C[\ln(R)]^3$$

Where:
- $T$ = absolute temperature (K)
- $R$ = resistance (Ω)
- $A$, $B$, $C$ = Steinhart-Hart coefficients (determined empirically)

**Characteristics:**
- **3 parameters** → higher accuracy over **wider temperature ranges** (~200°C span)
- Typical error: **±0.01°C to ±0.02°C** (vs. ±1°C for Beta)
- Standard in precision applications

---

## Inverse calculation (R from T)
### Beta equation
- Directly invertible:
  $$R = R_0 \exp\left[\beta\left(\frac{1}{T} - \frac{1}{T_0}\right)\right]$$

### Steinhart-Hart equation
- **No simple closed-form inversion** for $R(T)$ directly
- However, can be inverted using **Cardano's formula** by substitution:
  - Let $x = \ln(R)$
  - The equation becomes a **cubic in $x$**:
    $$Cx^3 + Bx + A - \frac{1}{T} = 0$$
  - Solve using Cardano's formula for depressed cubics
  - Then: $R = e^x$

---

## When to use which
| Model | Use Case |
|-------|----------|
| **Beta** | Cost-sensitive, narrow range (~±25°C from ref), moderate accuracy |
| **Steinhart-Hart** | Precision sensing, wide range, calibration data available |

---

## Key takeaway
**Beta** trades accuracy for simplicity; **Steinhart-Hart** adds one parameter for significantly better precision across wider ranges. Steinhart-Hart inversion is non-trivial but solvable analytically via **Cardano's formula** with the substitution $x = \ln(R)$.
