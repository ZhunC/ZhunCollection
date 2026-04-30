I read from aircraft control that they perform frequency sweep on each input during real flight to estimate system transfer function. how do they decide the freq sweep increment, and the min and max frequencies?

Great question! In aircraft control system identification, the frequency sweep parameters are chosen based on the aircraft dynamics and practical constraints:

## **Minimum Frequency (f_min)**

Determined by the **slowest modes** you want to identify:

- **Phugoid mode**: Typically 0.01-0.1 Hz (long-period oscillation in pitch/altitude)
- **Short period mode**: ~1-3 Hz
- **Dutch roll**: ~0.5-2 Hz
- Rule of thumb: Set f_min to **1/10 of the slowest mode frequency** you care about

## **Maximum Frequency (f_max)**

Limited by:

1. **Fastest dynamics**: Actuator bandwidth, structural modes (~10-30 Hz for large aircraft)
2. **Nyquist criterion**: Must be < 0.5 × sampling rate
3. **Pilot/passenger comfort**: Typically stay below 10-15 Hz
4. **Coherence degradation**: Beyond certain frequencies, signal-to-noise ratio drops

Typical range: **0.01 Hz to 10-20 Hz** for transport aircraft

## **Frequency Increment (Δf)**

Trade-offs:

- **Smaller Δf** → Better frequency resolution, longer test time
- **Larger Δf** → Faster test, but coarser resolution

Common approaches:

1. **Logarithmic spacing**: More points at low frequencies where dynamics change rapidly
    
    - `f_k = f_min × (f_max/f_min)^(k/N)` where k = 0,1,...,N
2. **Linear spacing**: Uniform `Δf = (f_max - f_min) / N`
    
3. **Mode-focused**: Denser spacing near known mode frequencies
    

**Typical values**: 20-50 frequency points over the range, giving Δf ≈ 0.2-0.5 Hz for linear spacing

## **Practical Considerations**

- **Dwell time per frequency**: 3-10 cycles minimum for good averaging
- **Total maneuver time**: Usually 60-300 seconds
- **Amplitude**: Large enough for SNR but within linear operating range (~5-10% of max deflection)