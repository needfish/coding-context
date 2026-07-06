# leda -- feedback control

## Problem

A system must keep a measured value (temperature, speed, voltage) close to a target (setpoint). It does this by repeatedly measuring the current value, comparing it to the target, and adjusting a control input. The rule for computing the adjustment (control law) changes per application.

## Input

- The setpoint (target value)
- The measured value (current reading from a sensor)
- The time since the last measurement (dt)

## Output

- A control signal (how much to adjust the actuator)

## Observable Behavior

- At each time step, the controller computes error = setpoint - measured
- The control signal is a function of the error, its history (integral), and its rate of change (derivative)
- The control signal is sent to the actuator (heater, motor, valve)
- Over time, the measured value converges toward the setpoint if the controller gains are tuned correctly
- If gains are too high, the output oscillates; if too low, convergence is slow

## Minimal Code

```python
class PIDController:
    def __init__(self, kp, ki, kd, setpoint=0):
        self.kp = kp
        self.ki = ki
        self.kd = kd
        self.setpoint = setpoint
        self.integral = 0
        self.last_error = 0

    def update(self, measured, dt):
        error = self.setpoint - measured
        self.integral += error * dt
        derivative = (error - self.last_error) / dt if dt > 0 else 0
        self.last_error = error
        return (self.kp * error) + (self.ki * self.integral) + (self.kd * derivative)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Integral windup | After a sustained disturbance, the integral term has grown large; when the disturbance ends, the output overshoots significantly | Clamp the integral term or freeze integration when the actuator is saturated |
| Oscillation | The output swings back and forth around the setpoint without settling | Reduce the proportional gain or add derivative damping |
| Sensor noise | Controller reacts to random measurement noise rather than actual process changes | Add a low-pass filter to the sensor reading |
| Sample jitter | Control loop does not run at a fixed frequency, causing inconsistent correction | Use a real-time timer or timestamp the samples |
| Phase lag | Sensor or actuator has a delay; controller corrects for past error, not current error | Add a predictor (Smith predictor) |

## Design Trade-offs

| Choice | What changes |
|---|---|
| P vs. PI vs. PID | P is simple but has steady-state error. PI eliminates steady-state error but can overshoot. PID reduces overshoot but is sensitive to noise. |
| Fixed vs. adaptive gains | Fixed is simple but must be tuned for one operating point. Adaptive adjusts to changing process dynamics but risks instability. |
| Continuous vs. on-off (bang-bang) | Continuous (PID) provides smooth correction. On-off (thermostat) is simple but oscillates around the setpoint. |

## Start With

Use a P controller first. Add integral term (PI) when steady-state error is a problem. Add derivative term (PID) when overshoot needs damping.

## Split Into Its Own Module When

- More than one control loop exists in the system.
- The control law needs tuning (gain scheduling, auto-tuning).
- The controller output must be logged and analysed.
- The control loop interacts with other control loops (cascaded control).

## Signs This Is Present

- A loop measures a value and adjusts an actuator based on an error.
- A PID, PI, or P controller is implemented.
- A value is compared to a target and the difference drives a corrective action.
- Hysteresis or deadband is used to prevent rapid switching.

## Common Mistakes

- Tuning the controller on a simulation and using the same gains in production (real dynamics differ).
- Not handling sensor failure (stale or NaN readings).
- Using the control interval as dt instead of the actual elapsed time.
- Starting the integral term from zero when the error is large (rapid saturation).
- Forgetting to clamp the controller output to the actuator limits.
