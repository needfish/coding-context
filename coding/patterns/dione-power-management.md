# ------------------------------------------------------------------------------
#  dione-power-management.md
#  Behavior variation -- the power policy algorithm changes per energy/performance trade-off.
# ------------------------------------------------------------------------------

# - Problem -

Your system needs to minimise energy consumption while staying responsive enough to handle external events. You need to trade off performance for power across CPU frequency, sleep states, peripheral gating, and communication duty cycling.

# - When You'll See This -

Battery-powered IoT devices, mobile phones, laptops, and energy-harvesting systems. Power is the primary constrained resource -- often more constrained than CPU cycles or memory. The system must react to events (sensor readings, network packets, user input) but spend the rest of its time in the deepest possible sleep state.

# - How It Works -

- **Energy source**: battery, supercapacitor, or energy harvester (solar, piezoelectric, RF) -- provides finite or intermittent power.
- **Power domains**: subsystems that can be independently powered -- CPU core, radio, sensor array, display, storage.
- **Operating states**: each domain has states with different power/performance trade-offs (active, idle, sleep, deep sleep, off).
- **Wake sources**: events that can transition a domain from sleep to active -- timer expiry, GPIO interrupt, network activity.

# - What Must Always Be True -

- The system spends as much time as possible in the lowest power state that still meets responsiveness requirements.
- Transitioning between states costs energy and time -- the benefit of sleeping must exceed the transition cost.
- All wake sources are configured before entering sleep.
- No domain is kept active unnecessarily.

# - What Goes Wrong -

- **Wake-up storm**: the system wakes up too frequently, spending more energy on transitions than it saves by sleeping.
- **Timer coupling**: timers are not aligned, so the system wakes up at many closely-spaced times instead of once.
- **Peripheral leak**: a sensor or radio stays powered on after use, draining the battery.
- **Transition overhead misestimation**: the energy cost of entering and leaving sleep exceeds the energy saved during sleep.
- **Worst-case provisioning**: the system is designed for peak power draw, but most of the time it needs much less.

# - Design Trade-Offs -

- **Sleep depth vs. wake latency**: deeper sleep saves more power but takes longer to wake up -- if wake latency exceeds event deadlines, you can't sleep that deep.
- **Aggressive vs. conservative**: aggressive power management saves more energy but risks missing events; conservative is safer but wastes power.
- **Centralised vs. distributed**: a central power manager has complete information but is a bottleneck; distributed decisions are faster but may conflict.

# - Related Problems -

- **pan/resource-scheduling**: power management is a scheduling problem over energy as the resource.
- **atlas/memory-allocation**: memory has power implications (more memory, more refresh power).
- **hyperion/network-listener**: networking is often the largest power consumer.

# - Getting Started -

Put the CPU to sleep in a loop when there's no work. Use the platform's idle/sleep instruction. No dynamic frequency scaling, no peripheral gating, no multiple power domains.

# - When to Split Into Its Own Module -

- You have multiple power domains that can be independently controlled.
- The system has battery life requirements that aren't being met.
- You need different power policies for different usage scenarios (performance vs. battery saver).
- The power management logic is scattered across drivers and application code.

# - Signs You're in This Pattern -

- Your device runs on a battery and you need to extend its life.
- You're setting CPU frequency or sleep states.
- You turn peripherals on and off based on usage.
- You measure energy consumption to optimise behaviour.

# - Common Mistakes -

- Sleeping too shallow (wasting power) because wake latency calculation is wrong.
- Sleeping too deep (missing events) because not all wake sources are configured.
- Not accounting for transition energy cost -- sleeping for very short periods wastes more energy than staying awake.
- Optimising CPU power while ignoring the radio or display, which dominate total power.

# - The Core Challenge -

Power management is a prediction problem. The system must decide whether to sleep or stay active based on an unknown future -- when will the next event arrive? How long will it take to process? The wrong prediction wastes energy or misses deadlines.
