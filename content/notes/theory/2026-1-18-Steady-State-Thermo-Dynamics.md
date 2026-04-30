---
title: Steady-Flow Water Heating Control Volume
tags: [thermodynamics, control-volume, steady-flow, energy-balance, fluids]
created: 2026-01-18
---

## System characterization
A **single-inlet / single-outlet open system (control volume)** with:
- Mass flow rate: **$\dot m$** (kg/s)
- Inlet state: **$(T_1, P_1)$**
- Outlet state: **$(T_2, P_2)$**
- Energy supplied to the flowing water: **$\dot Q$** (W), possibly time-varying **$\dot Q(t)$**
- Optional shaft work: **$\dot W_s$** (W) (e.g., pump/turbine). If none, set to 0.

## Assumptions (common)
- **Steady state**: no mass accumulation in the control volume.
- Often **negligible** kinetic and potential energy changes: $\Delta ke \approx 0$, $\Delta pe \approx 0$.
- Water may be treated as **incompressible liquid** if single-phase and not near boiling.

---

## Mass balance
General:
- $\dfrac{dm_{cv}}{dt} = \dot m_{in} - \dot m_{out}$

Steady-flow (most common):
- $\dot m_{in} = \dot m_{out} = \dot m$

---

## Energy balance (Steady-Flow First Law)
General steady-flow energy equation:
- $\dot Q - \dot W_s
= \dot m\left[(h_2-h_1) + \dfrac{V_2^2 - V_1^2}{2} + g(z_2-z_1)\right]$

Typical simplification (no shaft work, negligible KE/PE):
- $\dot Q \approx \dot m\,(h_2-h_1)$

---

## Property relations for water
### Single-phase liquid (common engineering approximation)
- $h_2-h_1 \approx c_p (T_2-T_1)$  
- Therefore: **$\dot Q \approx \dot m\,c_p\,(T_2-T_1)$**
- Use $c_p \approx 4180$ J/(kg·K) near room temperature (varies with $T$).

### If pressure effects are included (incompressible form)
- $dh \approx c_p\,dT + v\,dP$
- $h_2-h_1 \approx c_p(T_2-T_1) + v(P_2-P_1)$  
  (often $v\Delta P$ is small vs. $c_p\Delta T$ unless $\Delta P$ is large)

### If boiling / two-phase possible
- Do **not** use constant $c_p$ across phase change.
- Compute: $h_1=h(T_1,P_1)$, $h_2=h(T_2,P_2)$ using steam tables/IAPWS.

---

## Pressure drop interpretation
If $P_2 \ne P_1$:
- Often indicates **frictional losses / restrictions** (typically $P_2 < P_1$).
- A **pump** would cause $P_2 > P_1$ and introduces $\dot W_s$.

---

## Time-varying heat input
### Quasi-steady (fast response, little thermal storage)
- $\dot Q(t) \approx \dot m\,[h_2(t)-h_1(t)]$

### Dynamic with storage (device/water/metal thermal mass)
- $\dfrac{dU_{cv}}{dt} = \dot Q(t) + \dot m\,h_1 - \dot m\,h_2$
- With lumped liquid approximation: $U_{cv}\approx M c_p T_{cv}$ → yields a first-order model linking $\dot Q(t)$ to $T_2(t)$.

---

## Minimal “most-used” model
If single-phase liquid, no pump work, negligible KE/PE:
- **$\dot Q \approx \dot m\,c_p\,(T_2-T_1)$**
