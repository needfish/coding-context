# dione: Power Management

> **Pressure class:** Behavior variation  --  the power policy algorithm changes per energy/performance trade-off.

## Problem

A system must minimise energy consumption while maintaining sufficient responsiveness to external events, requiring dynamic tradeoffs between performance and power at multiple levels: CPU frequency, sleep states, peripheral gating, and communication duty cycling.

## Context

Arises in battery-powered IoT devices, mobile phones, laptops, and energy-harvesting systems. Power is the primary constrained resource  --  often more constrained than CPU cycles or memory. The system must react to external events (sensor readings, network packets, user input) but spend the rest of its time in the deepest possible sleep state. Each subsystem (CPU, radio, sensor, display) contributes to the power budget, and each has its own latency tradeoff for wakeup.

## Domains Involved

- **Energy source** (causal): battery, supercapacitor, energy harvester (solar, piezoelectric, RF)  --  provides finite or intermittent power.
- **Power domains** (lexical): independently gated regions of the chip (CPU core, peripheral bus, radio, memory retention) that can be powered off separately.
- **Sleep states** (lexical): graduated depth of power saving (clock gating  ->  voltage scaling  ->  power gating  ->  memory off), each with a wakeup latency and energy cost.
- **Duty cycle** (causal): the fraction of time a subsystem is active vs. asleep, typically expressed as a ratio or schedule.
- **Wakeup source** (causal): an event (pin change, timer, radio preamble, comparator threshold) that can bring the system out of sleep.
- **Scheduler** (causal): decides when to enter sleep and which sleep state to use based on predicted idle duration and pending work.
- **Frequency scaler** (causal): adjusts operating voltage and clock frequency (DVFS) to trade performance for power during active periods.

## Shared Phenomena

- Scheduler  ->  Power domain: sleep state entry command, wakeup timer configuration
- Sleep state  ->  Scheduler: wakeup interrupt, resume time, sleep duration measurement
- Frequency scaler  ->  CPU: voltage/frequency pair, transition completion notification
- Peripheral  ->  Scheduler: idle notification (no pending work), request for clock gating
- Radio  ->  Duty cycle: wake-up receiver signal, preamble detection, scheduled listen interval
- Energy source  ->  Scheduler: remaining capacity, voltage droop warning
- Application  ->  Scheduler: latency constraint, required wakeup deadline

## Invariants

- The system must wake from any sleep state within the maximum allowed latency for pending events.
- Transitioning between power states consumes energy; entering a deeper state is only beneficial if the idle duration exceeds the break-even time.
- All wakeup sources must be configured before entering sleep  --  an unconfigured source may cause the system to sleep indefinitely.
- Peripheral state (DMA descriptors, FIFO contents, register configurations) must be preserved across sleep or explicitly reinitialised on wake.
- The duty cycle must not exceed the available energy budget over the harvesting/charging interval.
- Frequency scaling must not cause the CPU to miss instruction-level deadlines  --  voltage must be sufficient for the chosen frequency.

## Known Failure Modes

- **Wakeup latency miss**: The system enters a deep sleep state, but an event arrives that requires faster wakeup, causing a missed ADC sample, lost radio packet, or user-perceptible delay.
- **Break-even violation**: Entering a deep sleep state for a very short idle period costs more energy than staying in a lighter state, due to wakeup transition energy.
- **State loss**: A deep sleep state powers off memory or peripheral state that was not saved, causing corruption on resume.
- **Frequency scaling lag**: The workload increases but the DVFS governor responds too slowly, causing performance degradation before frequency ramps up.
- **Energy cliff**: Battery voltage drops below the regulator's minimum, causing an abrupt system shutdown without clean state preservation.
- **Timer drift**: Low-power oscillators (RC, 32 kHz) drift with temperature, causing duty-cycle misalignment over long sleep periods.
- **Priority inversion in sleep**: A high-priority wakeup event arrives while the system is committing to sleep entry, and the wakeup handling is delayed until sleep entry completes.

## Forces

- **Power vs. latency**: Deeper sleep saves more power but takes longer to wake. The system must choose a sleep state whose wakeup latency is tolerable for the expected next event.
- **Active power vs. idle power**: DVFS reduces power when the CPU is active but less urgently; sleep states reduce power when the CPU is idle. The two strategies interact  --  a faster CPU finishes work sooner and can sleep longer.
- **Radio duty cycle vs. throughput**: Low duty cycling (e.g., 0.1% listen interval) saves battery but reduces throughput and increases end-to-end latency for network communication.
- **Hardware complexity vs. software control**: Dedicated power management hardware (PMIC, PWRCTRL) offloads sequencing but fixes the state machine; software control is flexible but requires careful timing and state tracking.
- **Harvesting vs. consuming**: Energy-harvesting systems must consume less power than the harvester supplies on average, but may burst above it using a storage buffer, requiring long-term energy accounting.

## Related Subproblems

- **pan/resource-scheduling**: Scheduling decisions (tickless idle, priority-based wakeup ordering) directly determine power consumption.
- **atlas/memory-allocation**: Memory retention across sleep states determines which regions can be powered off and which must stay alive.
- **saturn/routing**: Network routing decisions affect radio duty cycling  --  choosing a high-power neighbour shortens battery life.
- **hyperion/network-listener**: Network interfaces driver must coordinate wakeup timing with remote senders (e.g., TSCH slot scheduling, wake-on-LAN).
- **mars/persistence**: Power loss forces data to be committed to non-volatile storage before the energy reserve is exhausted.
- **europa/config-assembly**: Power management parameters (sleep thresholds, voltage tables, duty-cycle ratios) must be configured correctly for the hardware platform.

### ==== Practical Use ====

#### When this pattern matters
Any battery-powered or energy-harvesting device  --  IoT sensors, wearables, mobile phones, laptops, wireless nodes.

#### Start with (minimal form)
Run at full speed, never sleep. In embedded: `WFI` (wait for interrupt) in the idle loop. No DVFS, no tickless idle, no peripheral gating.

#### Split into its own module when (observable triggers)
- Battery life is measured and found insufficient (quantified gap between actual and required).
- The device spends most of its time idle  --  sleeping reduces average power measurably.
- Different operating modes have different performance requirements (active sensing vs. periodic reporting).
- The energy source is intermittent (harvesting) and consumption must track availability.

#### Signs you're in this pattern
- Your device runs on battery and battery life is a product requirement.
- Different operating modes consume significantly different power (active vs idle vs sleep).
- You need to trade off performance for battery life depending on remaining charge.
- The system generates heat that must be managed (thermal throttling).

#### Real project examples
- **smoltcp** (no explicit power management): Embedded TCP/IP stack designed for low-power MCUs  --  the pattern informs the design: poll-based operation avoids interrupt-driven wakeups.
- **zt terminal emulator** (framebuffer sleep): Terminal renders only on input or screen change  --  no idle refresh, display sleeps between updates.
- **tinykvm** (VM power management): KVM-based micro-VMs can be suspended/resumed  --  host manages VM power state via KVM API.

#### Common over-engineering traps
- Implementing DVFS before measuring that CPU active power dominates the power budget.
- Building a tickless scheduler before confirming that periodic timer wakeups drain the battery.
- Adding power-aware routing before measuring that the radio dominates the energy budget.

## Frame Concern

**The gap between energy consumption and energy availability.** The system consumes energy at different rates depending on its activity state, but the energy source provides power at a fixed or variable rate (battery discharge, solar harvesting). The gap widens when the system is active (burst consumption) and narrows when it sleeps (trickle charging). Managing power means predicting future energy availability and throttling consumption to stay within budget  --  a prediction problem at the physical level, not a scheduling problem.
