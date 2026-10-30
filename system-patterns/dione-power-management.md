# dione -- power management

## Problem

A system must use as little energy as possible while still doing its work on time. It can switch between power states (active, idle, sleep, off). The policy for when to switch states depends on workload, battery level, and thermal limits.

## Input

- Current workload level (active tasks, idle time)
- Battery level or power budget
- Thermal readings
- A set of available power states, each with wake-up latency and energy cost

## Output

- A power state transition command (go to idle, sleep, wake up)

## Observable Behavior

- The system transitions to a lower-power state when idle for long enough
- The system wakes up in response to events (timer, interrupt, GPIO)
- Waking from a lower-power state takes time; the time is known for each state
- Work is not lost during a sleep transition (state is saved or memory is kept refreshed)
- The system never enters a state from which it cannot wake

## Minimal Code

```python
import time

IDLE_THRESHOLD_MS = 100
SLEEP_DURATION_MS = 5000

def power_loop(hardware, events):
    last_active = time.monotonic()
    for event in events:
        if event == "work_done":
            elapsed = time.monotonic() - last_active
            if elapsed > IDLE_THRESHOLD_MS / 1000:
                hardware.sleep(SLEEP_DURATION_MS)
            last_active = time.monotonic()
        elif event == "wake":
            hardware.wake()
            last_active = time.monotonic()
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Wake-up storm | A peripheral generates interrupts as soon as the system enters sleep, preventing deep sleep | Debounce or mask spurious interrupts |
| Transition overhead dominates | System spends more energy entering/exiting sleep than staying active | Measure and tune the idle threshold |
| Priority inversion | A low-priority background task keeps the system awake, draining the battery | Use a wakelock or QoS mechanism |
| Thermal runaway | System stays active to process a burst and overheats, triggering throttling that slows processing | Proactive thermal scheduling |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Aggressive vs. conservative sleep | Aggressive saves more energy but risks latency violations. Conservative meets latency targets but wastes energy. |
| Hardware vs. software control | Hardware-managed states (CPU C-states) are fast but coarse. Software-managed states are flexible but slower. |
| Predictive vs. reactive | Predictive model sleeps earlier but may be wrong. Reactive (sleep after idle threshold) is simpler but wastes idle time. |

## Start With

Use a simple idle threshold: sleep when idle for longer than a fixed duration. Add predictive models (history-based, machine learning) when the workload is variable.

## Split Into Its Own Module When

- The system runs on battery and idle time exceeds 10% of runtime.
- Thermal throttling is triggered during normal operation.
- Multiple power states with different latency/energy trade-offs exist.
- The power management policy depends on the current workload, not just idle time.

## Signs This Is Present

- The system has multiple power states (active, idle, sleep, deep sleep, off).
- A governor or policy engine switches between power states based on load.
- Battery life or thermal output is a constraint.
- Wake-up sources are explicitly configured.

## Common Mistakes

- Sleeping too aggressively, causing missed deadlines or user-noticeable latency.
- Not accounting for transition energy: entering and leaving sleep costs energy too.
- Ignoring wake-up sources: the system cannot wake if wake-up sources are not configured.
- Using a single idle threshold for all workloads (interactive vs. batch need different tuning).
- Not handling spurious wake-ups (woken but no work to do).
