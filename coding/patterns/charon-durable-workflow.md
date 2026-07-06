# charon -- durable workflow

## Problem

A multi-step process (place order -> charge card -> ship -> send confirmation) must complete even if the program crashes or the machine fails mid-way through. After recovery, the process continues from where it left off, not from the beginning.

## Input

- A list of steps to execute in order
- Input data for the first step

## Output

- The final result after all steps complete successfully
- Or a failure state if a step fails and retries are exhausted

## Observable Behavior

- Steps execute in order, one after another
- After each step, the current state (which step completed, accumulated data) is saved to durable storage
- If the program crashes and restarts, it reads the saved state and continues from the last saved step
- If a step fails, it is retried (up to a configurable limit)
- If all retries are exhausted, the workflow enters a failed state
- Each step either completes fully or is retried -- no partial step results are visible

## Minimal Code

```python
import json, sqlite3, time

def run_workflow(steps, state_store):
    state = state_store.load()
    for step in steps[state.current_step:]:
        try:
            result = step.execute(state.data)
            state.current_step += 1
            state.data.update(result)
            state_store.save(state)
        except Exception as e:
            state.retry_count += 1
            if state.retry_count > 3:
                state.status = "failed"
                state.error = str(e)
                state_store.save(state)
                raise
            time.sleep(2 ** state.retry_count)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Non-idempotent step | Step runs twice (crash after execution but before state save), causing duplicate result | Make steps idempotent or use exactly-once semantics with deduplication |
| Step timeout | A step hangs forever, blocking the workflow | Set a timeout for each step; treat timeout as a failure |
| Infinite retry loop | A step always fails and the workflow retries forever | Cap retries and move to a failed terminal state |
| State corruption | A bug writes invalid workflow state to the durable store | Schema-version the state and test with property-based testing |
| Orphaned workflow | A workflow starts but never completes because its continuation was lost | Use a watchdog that detects idle workflows and restarts them |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Polling vs. event-driven | Polling (check for work every N seconds) is simple but wastes resources. Event-driven (step executor calls back when done) is efficient but needs a callback endpoint or message queue. |
| Centralised vs. distributed state | Single database for all state is simple but a bottleneck. Distributed log (Kafka, NATS) scales but adds complexity. |
| Compensation vs. rollback | Some steps have no rollback (email already sent). Use compensating actions instead of trying to undo. |
| Orchestration vs. choreography | One central workflow engine vs. each service emitting events that trigger the next step. Orchestration is simpler to reason about; choreography scales better. |

## Start With

Use a simple loop with state saved to a database after each step. Add retry with backoff for transient failures. Add a timeout for each step when steps can hang.

## Split Into Its Own Module When

- A multi-step process needs to survive process restarts.
- Manual recovery of failed processes is too slow or error-prone.
- The number of workflows exceeds what a single in-memory loop can handle.
- Workflow steps are distributed across multiple services.

## Signs This Is Present

- A sequence of operations is wrapped in try/except with retry logic.
- A crash mid-way through a process loses progress and requires manual recovery.
- A database is used to track whether each step of a multi-step job has completed.
- Callback handlers are needed to resume processing after an async operation.

## Common Mistakes

- Assuming steps are idempotent without verification.
- Storing all workflow state in memory and losing it on crash.
- Not handling step timeouts, causing workflows to hang forever.
- Using a single worker that cannot scale to more than one workflow at a time.
- Forgetting to handle the case where a step succeeds but the state write fails, causing the step to run again.
