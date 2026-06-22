# ------------------------------------------------------------------------------
#  pan-resource-scheduling.md
#  Behavior variation -- the scheduling policy algorithm changes per workload requirement.
# ------------------------------------------------------------------------------

# - Problem -

Multiple consumers compete for a shared resource (CPU time, network bandwidth, I/O channel, memory). Your system must allocate access according to a policy while avoiding starvation, priority inversion, and overload.

# - When You'll See This -

Any system with concurrent contention for a scarce resource -- OS task schedulers, I/O schedulers, network packet schedulers, database transaction schedulers, GPU command schedulers. Consumers have different urgency levels, and the scheduler must balance throughput, latency, and fairness.

# - How It Works -

- **Consumers**: tasks, threads, requests, packets, or transactions that need the resource.
- **Resource**: the thing they compete for -- CPU core, disk bandwidth, network link, memory arena.
- **Scheduling policy**: the rules that decide which consumer gets the resource next (priority, FIFO, round-robin, deadline).
- **Ready queue**: the set of consumers waiting for the resource.
- **Dispatcher**: the mechanism that hands the resource to the chosen consumer and preempts it when its turn ends.

# - What Must Always Be True -

- Every consumer eventually gets access to the resource (no starvation).
- The scheduling policy is followed consistently.
- Priority inversion is bounded or prevented (a low-priority task doesn't block a high-priority one indefinitely).
- The scheduler overhead is small compared to the resource's work time.
- Consumers that exceed their allocation are preempted or throttled.

# - What Goes Wrong -

- **Starvation**: a low-priority consumer never gets the resource because higher-priority ones keep arriving.
- **Priority inversion**: a high-priority task waits for a low-priority task that holds a shared lock.
- **Convoy effect**: one slow consumer holds the resource and forces others to wait in line behind it.
- **Priority inversion**: a task holds a lock needed by a higher-priority task.
- **Thundering herd**: waking all waiting consumers when the resource becomes available, overwhelming the system.
- **Deadline miss**: a real-time task misses its deadline because scheduling overhead is too high.

# - Design Trade-Offs -

- **Fairness vs. throughput**: giving everyone equal time is fair but may reduce total throughput; prioritising short tasks improves throughput but starves long ones.
- **Preemptive vs. cooperative**: preemptive scheduling (OS controls when tasks switch) is responsive; cooperative scheduling (tasks yield voluntarily) is simpler but can freeze the system.
- **Priority vs. deadline**: priority-based scheduling is simple but can cause starvation; deadline-based scheduling is more predictable but requires knowing deadlines in advance.

# - Related Problems -

- **atlas/memory-allocation**: memory allocation often needs scheduling for fair sharing.
- **hyperion/network-listener**: connection acceptance is a scheduling decision.
- **thebe/background-compaction**: compaction must not starve foreground reads and writes.

# - Getting Started -

A simple FIFO queue. First come, first served. No priorities, no preemption, no deadlines.

# - When to Split Into Its Own Module -

- You need priority levels for different consumers.
- Some consumers starve because higher-priority ones keep arriving.
- You need to bound maximum wait time (no task waits more than X seconds).
- Different consumers need different scheduling policies.

# - Signs You're in This Pattern -

- Multiple tasks or requests compete for the same resource.
- You have a queue with a dispatching loop.
- Some requests take much longer than others and delay everything.
- High-priority tasks wait behind low-priority ones.

# - Common Mistakes -

- Implementing priority scheduling without solving starvation (low priority tasks never run).
- Using a global lock for scheduling decisions on multi-core systems.
- Assuming all tasks are equally important and ignoring priority entirely.
- Not measuring scheduling overhead.

# - The Core Challenge -

The scheduler makes allocation decisions without knowing how long each consumer will need the resource. It must balance competing goals (fairness, throughput, latency, priority) based on partial information and changing conditions.
