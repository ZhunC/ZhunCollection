---
share: "True"
tags:
  - -signal_processing
postType: post
categories: Note
title: A Quick Adventure With CCMA
---



This is a quick note on the very interesting, wondrously constructed and powerful CCMA filter. You can find its official documentation here:
https://github.com/UniBwTAS/ccma.git

This quick note is available at:
https://colab.research.google.com/drive/1VYib7VSg8PKd71zckkgiGd_Uia5tjwtK?usp=sharing

# 1.  Signal Construction

The script generates a time series that contains:

- A low-frequency base sine wave
    `a * sin(2π f_base * t)
    
- A high-frequency noise sine wave with random frequency jitter  
    `c * sin( (hf + freq_noise(t)) * t )`
    
- Additive Gaussian noise
    
- A slow linear drift term
    

Together, these components simulate a realistic sensor signal containing:

- baseline oscillation
    
- high-frequency interference
    
- jitter in instantaneous frequency
    
- drift over time
    

This data is going to be used for evaluating CCMA and the baseline filter EMA.

---

# 2. CCMA Filtering

CCMA uses two window lengths:

- `w_ma` for the symmetric moving average component
    
- `w_cc` for a cascaded symmetric smoothing on its s-term
    

The resulting smoothing kernel is _symmetric_. This means the filter uses both past and future samples relative to time t.

Because of this symmetry, CCMA is originally **non-causal** and therefore not directly usable in real-time systems.

---

## 2.1 Making CCMA Causal

A symmetric filter has its effective center located at half the total kernel width:

`delay = (w_ma + w_cc) / 2 [in units of samples]

Shifting the filtered output forward by this delay makes the filter causal. The filtered value at time t now corresponds to the original CCMA output at time t + delay.

In the very early samples (where the shifted filter would require unavailable future data), the script fills with the raw signal.

**Observation:**  
The original CCMA can be converted into a real-time, causal filter simply by shifting the output by half the kernel width.

---

## 2.2 Tunability of CCMA

CCMA offers flexible control through its two window sizes:

- The moving average window adjusts coarse smoothing strength.
    
- The s-term filtering window modifies the smoothness and shape of the kernel.
    
- The combined effect allows fine-tuning of how aggressively high-frequency noise is suppressed.
    

This makes CCMA more adjustable than standard single-parameter smoothers.  
By adjusting the two window sizes, the filter can imitate behaviors ranging from a simple moving average to something closer to a Gaussian low-pass filter.

**Observation:**  
The combination of window sizes provides meaningful control over filter shape and strength, giving CCMA a useful tuning range.

---

# 3. EMA as a Baseline

EMA is included in the script not as a competitor but as a minimal baseline. It offers:

- a single tuning parameter (alpha = 0.05)
    
- causal behavior by design
    
- extremely low computational cost
    
- no external dependencies
    
- suitability for embedded or streaming systems
    

Despite its simplicity, EMA is often a reasonable first pass for smoothing due to its speed and predictable latency.

**Observation:**  
Because of speed, simplicity, zero dependencies, and low computation requirements, EMA remains a good baseline or first-pass smoothing method.

---

# 4. Summary of Key Points

- CCMA’s smoothing kernel is symmetric by default, making it non-causal.
    
- Shifting the CCMA output by half the combined window width produces a causal, real-time-ready version.
    
- CCMA offers flexible control over smoothing strength through its two window lengths.
    
- EMA is used as a baseline due to its simplicity and low cost, not as a replacement.
    
- Together, CCMA (shifted) and EMA provide a useful reference point for understanding smoothing behavior on noisy signals.