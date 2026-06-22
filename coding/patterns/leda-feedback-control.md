# ------------------------------------------------------------------------------
#  leda-feedback-control.md
#  Behavior variation -- the control algorithm changes per plant dynamics.
# ------------------------------------------------------------------------------

# - Problem -

You need to maintain a physical or logical variable at a desired value (setpoint) by measuring the current value, comparing it to the target, and adjusting an input to minimise the error over time -- while dealing with measurement noise, external disturbances, and changing conditions.

# - When You'll See This -

Any system that regulates a dynamic process. Temperature control (3D printer hotend, oven, HVAC), motor speed control (CNC spindle, drones), chemical process control (pH, flow rate), and software rate control (TCP congestion, rate limiters). The controller reads a sensor, computes the error, and applies a correction through an actuator.

# - How It Works -

- **Plant**: the process being controlled -- a motor, heater, vehicle, or network link.
- **Setpoint**: the target value for the controlled variable.
- **Sensor measurement**: the current value of the variable, possibly noisy or delayed.
- **Error**: the difference between setpoint and measured value.
- **Controller**: the algorithm that computes a correction from the error -- typically P, PI, or PID.
- **Actuator**: the mechanism that applies the correction to the plant.

# - What Must Always Be True -

- The controller reduces the error over time toward zero.
- The system remains stable (doesn't oscillate or diverge).
- The controller compensates for disturbances without manual adjustment.
- Sensor noise does not cause excessive actuator wear or instability.
- The controller respects actuator limits (can't command more than the actuator can deliver).

# - What Goes Wrong -

- **Integral windup**: the error accumulates while the actuator is saturated, so when the error reverses, the controller overcorrects.
- **Derivative kick**: a sudden change in setpoint causes a large derivative spike that makes the actuator jerk.
- **Oscillation**: the controller gains are too high, causing the system to overshoot and oscillate.
- **Sensor delay**: the measurement is delayed, so the controller acts on old data and overcorrects.
- **Noise amplification**: the derivative term amplifies sensor noise, causing erratic actuator movements.
- **Stability**: adding a controller to an otherwise stable plant can make it unstable.

# - Design Trade-Offs -

- **Aggressive vs. conservative tuning**: aggressive gains reach the setpoint fast but overshoot; conservative gains are stable but slow.
- **P-only vs. PI vs. PID**: P-only is simple but leaves steady-state error; PI eliminates steady-state error; PID adds predictive response but amplifies noise.
- **Fixed vs. adaptive gains**: fixed gains are simple but tuned for one operating point; adaptive gains adjust to changing plant dynamics but are much more complex.

# - Related Problems -

- **pan/resource-scheduling**: scheduling is a form of feedback control.
- **dione/power-management**: power management uses feedback to adjust performance states.

# - Getting Started -

A simple on/off (bang-bang) controller: if temperature < setpoint, turn heater on; if temperature > setpoint + hysteresis, turn heater off. No P, I, or D terms.

# - When to Split Into Its Own Module -

- On/off control causes too much oscillation or wears out the actuator.
- You need precise control around the setpoint (not just within a hysteresis band).
- The plant dynamics change with operating conditions (different speeds, different loads).
- You need to test the controller independently from the hardware.

# - Signs You're in This Pattern -

- You read a sensor, compute a difference from a target, and adjust an output to reduce the difference.
- Your control loop oscillates around the target.
- You're tuning parameters (P, I, D) to make the system behave correctly.
- The system overshoots the target and takes too long to settle.

# - Common Mistakes -

- Tuning PID gains by guesswork instead of using a systematic method (Ziegler-Nichols, Cohen-Coon).
- Implementing derivative on the error instead of on the measurement (derivative kick).
- Not clamping the integral term (integral windup).
- Sampling the sensor too slowly for the plant dynamics.

# - The Core Challenge -

A controller acts on imperfect information -- noisy sensors, delayed measurements, unknown disturbances. It must correct errors without knowing whether the error is real, transient, or measurement noise. The wrong correction makes things worse.
