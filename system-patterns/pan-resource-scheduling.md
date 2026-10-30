# pan -- resource scheduling

## Problem

Multiple tasks need the same limited resource (CPU, network bandwidth, disk I/O, memory). A scheduler decides which task gets the resource next and for how long. The decision policy (round-robin, priority, fair share, deadline) changes per use case.

## Input

- A list of tasks, each with a priority, deadline, or resource requirement
- The resource capacity (CPU cores, bandwidth, IOPS)

## Output

- Tasks are executed in the order determined by the scheduling policy
- Each task eventually runs and completes (unless the policy allows starvation)

## Observable Behavior

- Tasks are ordered by the scheduling policy (priority, deadline, fairness)
- A running task can be stopped (preempted) when a higher-priority task arrives or its time slice ends
- The preempted task resumes later from where it stopped
- The scheduler decides which task runs next in constant or logarithmic time per decision

## Minimal Code

```python
from heapq import heappush, heappop

class Scheduler:
    def __init__(self):
        self.queue = []
        self.counter = 0

    def submit(self, task, priority=0):
        heappush(self.queue, (-priority, self.counter, task))
        self.counter += 1

    def run_next(self):
        if not self.queue:
            return None
        _, _, task = heappop(self.queue)
        return task()
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Starvation | A low-priority task never runs because higher-priority tasks keep arriving | Use aging (increase priority over time) or fair scheduling |
| Priority inversion | A high-priority task waits for a resource held by a low-priority task, and a medium-priority task runs instead | Use priority inheritance (the low-priority task inherits high priority temporarily) |
| Convoy effect | A long-running task holds a resource, all other tasks queue behind it | Use preemptive scheduling with time slices |
| Thrashing | Scheduler spends more time switching between tasks than running them | Limit the number of active tasks or increase the time slice |
| Deadline miss | A task finishes after its deadline because the scheduler did not prioritise it in time | Use Earliest Deadline First scheduling for hard real-time tasks |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Preemptive vs. cooperative | Preemptive (time slice) ensures fairness but adds context-switch overhead. Cooperative (tasks yield voluntarily) has lower overhead but a misbehaving task can block the system. |
| Priority vs. fairness | Priority ensures important tasks run first but can starve low-priority tasks. Fair (round-robin, CFS) ensures all tasks make progress but may delay high-priority tasks. |
| Static vs. dynamic priority | Static priority (set once) is simple but does not adapt. Dynamic priority (aging, deadline-driven) adapts but adds complexity. |

## Start With

Use a priority queue (heap) for priority scheduling or a FIFO queue for round-robin. Add preemption when tasks can run longer than their allowed time slice.

## Split Into Its Own Module When

- More than one resource type is scheduled (CPU + I/O + network).
- Multiple scheduling policies are needed (different tasks need different policies).
- Scheduling decisions must be logged and audited.
- The scheduler must handle preemption (suspend and resume tasks).

## Signs This Is Present

- A queue of tasks is maintained with ordering logic.
- Priority, deadline, or round-robin logic is implemented.
- Concurrency limits or rate limiters are used.
- A thread pool or task queue is used with ordering rules.

## Common Mistakes

- Not handling priority inversion (use priority inheritance or priority ceiling).
- Using a busy-wait loop instead of a blocking queue.
- Assuming all tasks are equal in priority or resource requirements.
- Not accounting for scheduling overhead (context switching, queue operations).
- Forgetting to handle task cancellation (remove from queue, clean up resources).
