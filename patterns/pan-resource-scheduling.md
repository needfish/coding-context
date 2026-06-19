# pan: Resource Scheduling

> **Pressure class:** Behavior variation  --  the scheduling policy algorithm changes per workload requirement.

## Problem

Multiple consumers compete for a shared resource (CPU, network bandwidth, memory bandwidth, I/O channel), and the system must allocate access according to a policy (priority, fairness, deadline) while avoiding starvation, priority inversion, and overload.

## Context

Arises in any system with concurrent contention for a scarce resource: RTOS task schedulers, I/O schedulers, network packet schedulers, GPU command schedulers, database transaction schedulers, and disk schedulers. The resource is typically single-unit but can be partitioned (e.g., CPU time slices, disk timeslots). Consumers have different urgency levels, and the scheduling policy must balance throughput, latency, and fairness.

## Domains Involved

- **Consumers** (causal): tasks, threads, processes, I/O requests, packets, or transactions that need access to the resource.
- **Resource** (causal): the shared entity (CPU core, network link, disk, bus) that can serve one consumer at a time (or N consumers in parallel, with N fixed).
- **Scheduler** (causal): the policy function that selects which consumer to serve next, given their priorities, deadlines, and past service.
- **Ready queue** (lexical): the ordered data structure holding consumers that are eligible to run.
- **Priority** (lexical): an urgency level assigned to each consumer, either statically or dynamically.
- **Deadline** (lexical): a time by which a consumer must receive service to meet its requirement.
- **Preemption** (causal): the ability to suspend a running consumer and switch to a higher-priority one.

## Shared Phenomena

- Consumers  ->  Scheduler: arrival events, priority/deadline metadata, state transitions (ready, blocked, suspended)
- Scheduler  ->  Resource: dispatch decisions (which consumer to serve next)
- Resource  ->  Scheduler: completion events, preemption interrupts
- Scheduler  ->  Ready queue: enqueue/dequeue operations
- Scheduler  ->  Consumers: context switch, priority boost, starvation detection

## Invariants

- At most one consumer may occupy the resource at any instant (for single-core scheduling).
- A higher-priority consumer must never wait indefinitely behind a lower-priority one (priority ordering).
- The scheduler must make progress  --  if any consumer is ready, the scheduler must eventually select one.
- A consumer that releases the resource must be able to re-enter the ready queue (no permanent starvation by policy).
- Preemption must save and restore consumer state completely (register file, program counter, stack pointer).
- Time consumed by the scheduler itself must be bounded and accounted for (scheduling overhead).

## Known Failure Modes

- **Priority inversion**: A low-priority consumer holds a resource needed by a high-priority consumer, causing the high-priority one to wait. Mitigated by priority inheritance or priority ceiling protocols.
- **Starvation**: A consumer never receives service because higher-priority consumers continuously arrive. Mitigated by ageing (gradually increasing priority of starved consumers) or fair queuing.
- **Convoy effect**: A long-running consumer blocks behind a short one, or vice versa in FIFO scheduling, causing bursty latency.
- **Thundering herd**: Multiple high-priority consumers become ready simultaneously and the scheduler cannot distinguish which to run first, causing jitter.
- **Scheduler thrashing**: The overhead of scheduling decisions exceeds the useful work done, typically when time slices are too short or context switches are too expensive.
- **Deadline miss cascade**: One consumer misses its deadline, causing dependent consumers to also miss theirs, amplifying failure.

## Forces

- **Throughput vs. latency**: Batch-oriented scheduling (FIFO, round-robin) maximises throughput but can starve latency-sensitive consumers; preemptive priority scheduling minimises latency for critical consumers but adds overhead and can reduce throughput.
- **Fairness vs. urgency**: Fair schedulers (proportional share, lottery) give each consumer a guaranteed fraction; priority schedulers favour urgent work but can starve low-priority consumers.
- **Preemptive vs. cooperative**: Preemptive scheduling guarantees responsiveness but requires atomic context switching and shared-data protection; cooperative scheduling is simpler but requires consumers to voluntarily yield, risking unbounded latency.
- **Static vs. dynamic priority**: Static priorities are simple but cannot adapt to changing workloads; dynamic priorities (EDF, deadline-driven) adapt optimally but require knowledge of deadlines.
- **Global vs. partitioned**: Global scheduling allows any consumer on any core but suffers from cache migration overhead; partitioned scheduling pins consumers to cores but loses load-balancing flexibility on multi-core.

## Related Subproblems

- **atlas/memory-allocation**: Memory allocation routines must be reentrant and limited in duration  --  they share the CPU with application code and must not cause unbounded scheduling delay.
- **dione/power-management**: Tickless idle and frequency scaling couple scheduling decisions with energy consumption.
- **mars/persistence**: I/O scheduling (disk, flash) must account for durability and wear-leveling constraints alongside latency.
- **jupiter/validation**: Schedulers often include watchdog timers and deadline monitors as validation of correct scheduling.
- **uranus/authentication**: Real-time systems must authenticate task origins before scheduling critical work.
- **hyperion/network-listener**: Network interrupt handling must be scheduled alongside application tasks.

### ==== Practical Use ====

#### When this pattern matters
Any system with concurrent contention for a shared resource: RTOS task schedulers, I/O schedulers, network packet schedulers, request rate limiters.

#### Start with (minimal form)
First-come, first-served (FIFO queue). A single-threaded event loop. No priorities, no preemption, no deadlines. Python: `asyncio.Queue`. Rust: `std::collections::VecDeque`.

#### Split into its own module when (observable triggers)
- Some work items are more urgent than others (priorities emerge).
- A high-priority item can arrive while a low-priority item is being processed (preemption needed).
- Items have deadlines, and missing them causes real consequences.
- The scheduling policy needs to be configurable (FIFO vs priority vs round-robin vs EDF).

#### Signs you're in this pattern
- Multiple tasks compete for a shared resource (CPU time, network bandwidth, disk I/O).
- Some tasks have deadlines or quality-of-service requirements.
- Tasks arrive unpredictably and you need to decide which runs next.
- A naive FIFO or round-robin causes some tasks to starve or miss deadlines.

#### Real project examples
- **Flowstate** (`delay.go`): Delayed step execution  --  schedules workflow steps to run after a timeout or at a specific time using a timer heap.
- **tiny-lsm-go** (compaction scheduler): Background compaction tasks scheduled when write amplification crosses a threshold  --  priority to level-0 compaction to bound read amplification.
- **NessMuxer** (packet scheduling): Video packets from multiple streams are scheduled for muxing based on PTS timestamps  --  deadline-driven scheduling.
- **smoltcp** (socket polling): Network sockets are polled round-robin with fairness guarantees  --  prevents a single socket from starving others.

#### Common over-engineering traps
- Implementing priority queues before confirming that FIFO causes missed deadlines.
- Building a preemptive scheduler when cooperative yielding handles all cases.
- Adding deadline-monitoring infrastructure before a single deadline has been missed.

## Frame Concern

**The gap between priority assignment and actual timeliness.** The scheduler assigns priority to consumers, but the actual time a consumer waits depends on the interaction between priorities, resource holding times, blocking on shared resources, and preemption patterns. A consumer with high priority can still miss its deadline if it shares a resource with a lower-priority consumer (priority inversion) or if interrupt handling delays its activation. The concern is how the scheduler's policy maps to the actual temporal behaviour experienced by consumers  --  policy is abstract, but timeliness is concrete.
