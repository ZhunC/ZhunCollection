# PID Controllers for Double Pendulum: Fast Prototyping & Space-Economic Design

## Introduction

PID (Proportional-Integral-Derivative) controllers are perhaps the most widely adopted controller. A simple, easy to implement calculation represents fundamental understanding from its inventors to dynamic systems. I want to use this post to record and discuss my understanding to PIDs.

**Double Pendulum System**: Our example system consists of two coupled pendulums, representing a nonlinear underactuated system in a simple form. 

## System Model

The double pendulum dynamics can be linearized around the upright equilibrium position:

```
State vector: x = [θ₁, θ₁̇, θ₂, θ₂̇]ᵀ
Control input: u (torque applied to first joint)

Linear state-space representation:
ẋ = Ax + Bu
y = Cx + Du
```

Where the system exhibits inherent instability requiring active stabilization.

## Stability Guarantee

### Theoretical Foundation

For PID controllers applied to the linearized double pendulum, stability can be analyzed using the characteristic equation:

```
det(sI - A + BKc) = 0
```

Where Kc = [Kp + Ki/s + Kds] represents the PID compensation.

**Key Stability Considerations:**

1. **Proportional Gain (Kp)**: Provides immediate response to error but can introduce steady-state offset
2. **Integral Gain (Ki)**: Eliminates steady-state error but may cause overshoot and oscillations
3. **Derivative Gain (Kd)**: Provides damping and anticipatory action but amplifies noise

**Stability Margins**: The double pendulum's open-loop instability requires careful tuning to ensure:

- Phase margin > 45° for robust performance
- Gain margin > 6dB to handle model uncertainties
- Closed-loop bandwidth sufficient for disturbance rejection

### Practical Implementation

```python
class PIDController:
    def __init__(self, kp, ki, kd, dt, output_limits=(-10, 10)):
        self.kp, self.ki, self.kd = kp, ki, kd
        self.dt = dt
        self.output_limits = output_limits
        self.reset()
    
    def reset(self):
        self.integral = 0
        self.previous_error = 0
    
    def compute(self, setpoint, measurement):
        error = setpoint - measurement
        self.integral += error * self.dt
        derivative = (error - self.previous_error) / self.dt
        
        output = (self.kp * error + 
                 self.ki * self.integral + 
                 self.kd * derivative)
        
        # Anti-windup
        output = max(min(output, self.output_limits[1]), self.output_limits[0])
        self.previous_error = error
        return output
```

## Parameter Tuning

### Ziegler-Nichols Method (Modified for Double Pendulum)

The classical Ziegler-Nichols approach requires adaptation for the double pendulum's complex dynamics:

1. **Ultimate Gain Method**:
    
    - Set Ki = Kd = 0
    - Increase Kp until sustained oscillation occurs (Ku)
    - Measure oscillation period (Tu)
    - Apply tuning rules: Kp = 0.6Ku, Ki = 2Kp/Tu, Kd = KpTu/8
2. **Modified Tuning for Double Pendulum**:
    
    - Conservative factors due to nonlinearity: Kp = 0.4Ku
    - Reduced integral action: Ki = 1.5Kp/Tu
    - Enhanced derivative action: Kd = 1.2KpTu/8

### Manual Tuning Strategy

For rapid prototyping, a systematic manual approach:

1. **Start with P-only control**: Tune Kp for acceptable rise time
2. **Add integral action**: Increase Ki until steady-state error is eliminated
3. **Add derivative action**: Tune Kd to reduce overshoot and improve stability
4. **Iterate**: Fine-tune based on performance requirements

### Auto-tuning Implementation

```python
def relay_feedback_tuning(plant_model, amplitude=1.0, max_iterations=50):
    """
    Relay feedback auto-tuning for double pendulum PID parameters
    Returns: (Ku, Tu) - Ultimate gain and period
    """
    relay_output = [-amplitude, amplitude]
    oscillation_data = []
    
    for i in range(max_iterations):
        # Implement relay feedback test
        # Measure oscillation characteristics
        pass
    
    # Calculate ultimate parameters
    Ku = 4 * amplitude / (np.pi * oscillation_amplitude)
    Tu = measured_period
    
    # Return conservative PID parameters
    return {
        'Kp': 0.4 * Ku,
        'Ki': 1.5 * 0.4 * Ku / Tu,
        'Kd': 1.2 * 0.4 * Ku * Tu / 8
    }
```

## Pole Placement

### Design Methodology

For the double pendulum system, pole placement allows precise specification of closed-loop dynamics:

**Desired Pole Locations**:

- Fast poles for pendulum stabilization: s = -10 ± 5j
- Slower poles for smooth reference tracking: s = -2, -3

### PID-based Pole Placement

Unlike full state feedback, PID controllers provide limited pole placement capability. However, we can approximate desired performance:

```python
def pid_pole_placement(A, B, desired_poles):
    """
    Approximate pole placement using PID structure
    """
    # Convert desired poles to characteristic polynomial
    char_poly = np.poly(desired_poles)
    
    # Extract PID gains that approximate desired response
    # This is a simplified approach - full implementation requires
    # numerical optimization
    
    return kp_approx, ki_approx, kd_approx
```

### Limitations and Workarounds

PID controllers cannot achieve arbitrary pole placement for higher-order systems like the double pendulum. Workarounds include:

1. **Cascade Control**: Inner/outer loop structure
2. **Feedforward Compensation**: Precomputed control signals
3. **Gain Scheduling**: Parameter adaptation based on operating point

## LQR (Linear Quadratic Regulator)

### Comparison with PID

While PID excels in simplicity, LQR provides optimal performance for known system models:

**LQR Cost Function**:

```
J = ∫₀^∞ (xᵀQx + uᵀRu) dt
```

**Advantages of LQR over PID**:

- Guaranteed stability margins (60° phase, ∞ gain margin)
- Optimal performance for given cost weights
- Systematic design procedure

**PID Advantages over LQR**:

- No state estimation required
- Robust to model uncertainties
- Simpler implementation and tuning

### Hybrid Approach: LQR-informed PID Tuning

Use LQR solution to guide PID parameter selection:

```python
def lqr_informed_pid(A, B, Q, R):
    """
    Use LQR solution to suggest PID parameters
    """
    # Solve LQR
    K_lqr = lqr_solve(A, B, Q, R)
    
    # Map to PID structure (approximate)
    # This requires careful analysis of the closed-loop system
    kp_estimate = K_lqr[0, 0]  # Position feedback
    kd_estimate = K_lqr[0, 1]  # Velocity feedback
    ki_estimate = calculate_ki_from_steady_state_requirements()
    
    return kp_estimate, ki_estimate, kd_estimate
```

## Cancellation and Feedforward Terms

### Zero-Pole Cancellation

The double pendulum may have zeros that limit performance. PID design should account for:

1. **Right-half-plane zeros**: Cannot be canceled (non-minimum phase)
2. **Slow zeros**: May be canceled with caution
3. **Fast zeros**: Generally safe to cancel

### Feedforward Compensation

For reference tracking, feedforward significantly improves PID performance:

```python
class PIDWithFeedforward:
    def __init__(self, pid_controller, feedforward_gains):
        self.pid = pid_controller
        self.ff_gains = feedforward_gains
    
    def compute(self, reference, measurement, reference_derivative=0):
        # Feedback component
        fb_output = self.pid.compute(reference, measurement)
        
        # Feedforward component
        ff_output = (self.ff_gains['proportional'] * reference +
                    self.ff_gains['velocity'] * reference_derivative)
        
        return fb_output + ff_output
```

### Disturbance Feedforward

For the double pendulum, measurable disturbances (e.g., external forces) can be compensated:

```python
def disturbance_feedforward(disturbance, plant_model):
    """
    Compute feedforward control to reject known disturbances
    """
    # Calculate required control effort to counteract disturbance
    return -np.linalg.pinv(plant_model.B) @ disturbance
```

## Time and Space Complexity Analysis

### Computational Complexity

**PID Controller**:

- **Time Complexity**: O(1) per sample - constant time execution
- **Space Complexity**: O(1) - minimal memory requirements (3 gains + 2 state variables)

**Comparison with Advanced Controllers**:

|Controller Type|Time Complexity|Space Complexity|Memory Usage|
|---|---|---|---|
|PID|O(1)|O(1)|~20 bytes|
|LQR|O(n²)|O(n²)|~n²×8 bytes|
|MPC|O(n³N)|O(nN)|~nN×8 bytes|
|Neural Network|O(nm)|O(nm)|~nm×4 bytes|

Where n = state dimension, N = prediction horizon, m = number of neurons.

### Real-time Performance

**Execution Time Analysis** (for double pendulum, n=4):

```python
import time

def benchmark_controllers():
    # PID execution time
    start = time.perf_counter()
    for _ in range(10000):
        pid_output = pid.compute(setpoint, measurement)
    pid_time = (time.perf_counter() - start) / 10000
    
    # LQR execution time
    start = time.perf_counter()
    for _ in range(10000):
        lqr_output = K_lqr @ state_estimate
    lqr_time = (time.perf_counter() - start) / 10000
    
    print(f"PID: {pid_time*1e6:.2f} μs per sample")
    print(f"LQR: {lqr_time*1e6:.2f} μs per sample")
```

**Typical Results**:

- PID: ~1-5 μs per sample
- LQR: ~10-50 μs per sample
- MPC: ~1-10 ms per sample

### Memory Footprint

**PID Controller Memory Layout**:

```c
typedef struct {
    float kp, ki, kd;          // 12 bytes
    float integral;            // 4 bytes
    float previous_error;      // 4 bytes
    float dt;                  // 4 bytes
    float output_limits[2];    // 8 bytes
} PIDController;              // Total: 32 bytes
```

**Embedded Implementation Considerations**:

1. **Fixed-point arithmetic**: Reduce memory and computation
2. **Anti-windup schemes**: Prevent integral saturation
3. **Derivative filtering**: Reduce noise sensitivity
4. **Bumpless transfer**: Smooth parameter changes

### Scalability Analysis

**Multi-loop PID Systems**:

- **Cascade control**: 2-3 PID loops, ~100 bytes total
- **Decentralized control**: n independent PIDs, ~32n bytes
- **Coordinated control**: Additional logic, ~50n bytes

**Performance vs. Resource Trade-offs**:

```python
def control_system_comparison():
    systems = {
        'PID': {'performance': 7, 'complexity': 2, 'memory': 1},
        'PID+FF': {'performance': 8, 'complexity': 3, 'memory': 2},
        'LQR': {'performance': 9, 'complexity': 6, 'memory': 5},
        'MPC': {'performance': 10, 'complexity': 9, 'memory': 8}
    }
    
    # Calculate efficiency metric
    for system, metrics in systems.items():
        efficiency = metrics['performance'] / (metrics['complexity'] + metrics['memory'])
        systems[system]['efficiency'] = efficiency
    
    return systems
```

## Rapid Prototyping Guidelines

### Development Workflow

1. **System Identification**: Simple step response tests
2. **Initial Tuning**: Conservative Ziegler-Nichols parameters
3. **Performance Testing**: Closed-loop response evaluation
4. **Iterative Refinement**: Manual fine-tuning based on requirements
5. **Robustness Validation**: Parameter sensitivity analysis

### Implementation Best Practices

```python
class RobustPIDController:
    def __init__(self, kp, ki, kd, dt):
        self.kp, self.ki, self.kd, self.dt = kp, ki, kd, dt
        self.reset()
        
    def reset(self):
        self.integral = 0
        self.previous_error = 0
        self.derivative_filter = 0
        
    def compute(self, setpoint, measurement):
        error = setpoint - measurement
        
        # Anti-windup integral
        if abs(self.integral) < 100:  # Saturation limit
            self.integral += error * self.dt
            
        # Filtered derivative
        derivative_raw = (error - self.previous_error) / self.dt
        alpha = 0.1  # Filter coefficient
        self.derivative_filter = (alpha * derivative_raw + 
                                 (1 - alpha) * self.derivative_filter)
        
        output = (self.kp * error + 
                 self.ki * self.integral + 
                 self.kd * self.derivative_filter)
        
        self.previous_error = error
        return output
```

### Testing and Validation

**Performance Metrics for Double Pendulum**:

- **Settling time**: < 2 seconds for small disturbances
- **Overshoot**: < 10% for step references
- **Steady-state error**: < 1% with integral action
- **Disturbance rejection**: Return to setpoint within 3 seconds

## Conclusion

PID controllers offer an optimal balance of performance, simplicity, and resource efficiency for double pendulum control. Their constant-time execution (O(1)) and minimal memory footprint (~32 bytes) make them ideal for embedded applications and rapid prototyping scenarios.

Key advantages for space-economic design:

- Minimal computational overhead
- No state estimation requirements
- Robust performance across operating conditions
- Simple parameter tuning procedures
- Extensive industrial validation and support

While advanced controllers like LQR and MPC may offer superior theoretical performance, PID controllers remain the practical choice when development time, computational resources, and implementation complexity are primary concerns.