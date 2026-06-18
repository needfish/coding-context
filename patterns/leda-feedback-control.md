# leda: Feedback Control

## Problem

A system must maintain a physical or logical variable at a desired setpoint by measuring the current value, comparing it to the target, and adjusting an actuator input to minimise the error over time — all while coping with measurement noise, external disturbances, actuator limits, and changing dynamics.

## Context

Arises in any system that regulates a dynamic process: temperature control (3D printer hotend, oven, HVAC), motor speed/position control (CNC spindle, quadcopter motors), chemical process control (pH, flow rate), and software rate/throttle control (TCP congestion, rate limiters). The controller reads a sensor, computes error = setpoint − measured, and applies a correction through an actuator. The physical plant has inertia, delay, and nonlinearities that the controller must compensate for.

## Domains Involved

- **Plant** (causal): the physical or logical process being controlled (motor, heater, vehicle, network link).
- **Setpoint** (lexical): the desired value of the controlled variable (target temperature, target RPM, target altitude).
- **Sensor** (causal): measures the actual value of the controlled variable, with noise, bias, and finite resolution.
- **Controller** (causal): computes the control signal from the error, typically using proportional, integral, and derivative terms (PID) or a state-space law.
- **Actuator** (causal): applies the control signal to the plant (PWM voltage, servo position, throttle percentage).
- **Disturbance** (causal): external influences on the plant that the controller must reject (wind gust, load change, ambient temperature shift).
- **Error** (lexical): the difference between setpoint and measured value at a given instant.

## Shared Phenomena

- Setpoint → Controller: target value
- Sensor → Controller: measured value (with noise, delay)
- Controller → Actuator: control signal (PWM duty, voltage, force)
- Actuator → Plant: physical input
- Plant → Sensor: actual state of the controlled variable
- Disturbance → Plant: unmeasured external influence

## Invariants

- The control signal must remain within actuator limits (saturation) — integral windup must be prevented when the actuator is saturated.
- The controller must be stable: bounded input produces bounded output, and the error must converge to zero (or a bounded neighbourhood) under constant setpoint.
- The sensor sample rate must exceed twice the highest-frequency component of the plant dynamics (Nyquist criterion) to avoid aliasing.
- The combined phase lag of sensor, controller computation, and actuator response must not exceed the stability margin.
- Integral windup must be prevented: the integral term must not accumulate when the actuator is saturated.

## Known Failure Modes

- **Limit cycle oscillation**: The controller causes the plant to oscillate between two states (e.g., temperature overshoots setpoint, heater turns off, temperature drops below, heater turns on) due to relay-like on/off control without hysteresis.
- **Integral windup**: The integral term accumulates while the actuator is saturated, causing a large overshoot when the actuator desaturates.
- **Derivative kick**: A sudden change in setpoint causes the derivative term to spike, sending the actuator to its limit.
- **Sensor noise amplification**: The derivative term amplifies high-frequency sensor noise, causing jittery actuator output.
- **Phase margin violation**: Excess delay in the loop causes the controller to fight its own previous correction, leading to growing oscillations.
- **Estimation lag**: When the controller relies on a state estimator (observer, Kalman filter), the estimator's lag can cause the controller to act on stale information, reducing stability margin.
- **Gain scheduling mismatch**: If controller gains are scheduled based on operating point, a rapid transition between regimes can leave mismatched gains temporarily active.

## Forces

- **Aggressiveness vs. stability**: High proportional gain reduces error quickly but can cause oscillation; low gain is stable but slow to respond.
- **Noise rejection vs. responsiveness**: Heavy filtering of sensor noise adds phase lag, reducing the controller's ability to reject high-frequency disturbances; light filtering passes noise to the actuator.
- **Integral action vs. windup**: Integral action eliminates steady-state error but accumulates during saturation; windup protection (clamping, back-calculation) adds complexity.
- **Model-based vs. model-free**: Model-based control (LQR, pole placement) requires accurate plant dynamics but can be optimised; PID requires no model but cannot anticipate future disturbances.
- **Fixed vs. adaptive gains**: Fixed gains are simple but must be conservatively tuned for worst-case dynamics; adaptive gains track changing plant dynamics but risk instability if adaptation is too fast.

## Related Subproblems

- **pan/resource-scheduling**: The controller runs as a periodic task and must complete its computation within the control interval — scheduling latency directly affects control stability.
- **jupiter/validate**: Sensor validation (outlier rejection, plausibility checks) prevents bad measurements from corrupting the controller state.
- **neptune/transform**: The actuator mixer transforms logical control signals (roll/pitch/yaw/thrust) into physical actuator outputs (motor PWMs).
- **mars/persist**: State estimators (EKF, complementary filter) maintain a persistent estimate of system state across sensor dropout.
- **dione/power-management**: Power-aware controllers reduce actuator output to conserve energy, trading control accuracy for power savings.

## Frame Concern

**The gap between measurement and truth.** The controller acts on sensor measurements, not the actual state of the plant. Every sensor has noise, bias, delay, and finite resolution — the controller cannot directly know the true state. It must infer it from imperfect measurements and compensate for the gap through filtering, estimation, and robust gain margins. When the gap grows too large (sensor failure, unmodelled dynamics, excessive delay), the controller's corrections become counterproductive and the system destabilises.
