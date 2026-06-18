# charon: Durable Workflow / Saga

> **Pressure class:** Behavior variation — the workflow step sequence changes per business process.

## Problem

A multi-step process must make progress across process restarts, partial failures, and arbitrary delays, ensuring that each step is executed exactly once and the overall process eventually reaches a terminal state (success or failure) without manual intervention.

## Context

Arises in any system with long-running business processes that span multiple operations, services, or time boundaries: order fulfilment, payment processing, data migration pipelines, deployment rollouts, document approval chains, and multi-step ETL jobs. The process is decomposed into discrete steps, each with a well-defined input and output. The system must persist the progress between steps so that a crash or restart picks up where it left off. Steps may have compensating actions (rollbacks) that must be run if a later step fails.

## Domains Involved

- **Workflow definition** (lexical): the static description of the process — a graph of steps with transitions, retry policies, timeouts, and compensating actions.
- **Execution state** (lexical): the current position in the workflow — which step is active, what data has been accumulated, what side effects have been applied.
- **Step handler** (causal): the function or service that implements a single step of the workflow. May call external services, perform computation, or schedule future work.
- **State store** (causal): the persistent storage that holds execution state across restarts. Must be transactional: step completion and state update must be atomic.
- **Recovery controller** (causal): the process that scans for incomplete workflows on startup and resumes them.
- **Compensation handler** (causal): the inverse of a step handler — undoes the side effects of a completed step when a later step fails.

## Shared Phenomena

- Step handler → State store: read current state, write new state (atomically with step completion)
- Workflow definition → Engine: step graph, transition rules, retry policy
- Engine → Step handler: input data, context, deadline metadata
- State store → Recovery controller: incomplete workflows on startup
- Recovery controller → Engine: resume workflows, retry failed steps
- Engine → Compensation handler: rollback request with original step output

## Invariants

- The execution state must be persisted atomically with the step completion — no step is "done" until its state is safely stored.
- A workflow must eventually reach a terminal state (completed, failed, or compensated) — no infinite pending states.
- A step must be executed at most once (idempotency) or its side effects must be idempotent.
- If a workflow fails after some steps have completed, compensating actions must be run in reverse order for all completed steps.
- Two recovery controllers must not execute the same workflow step concurrently (exclusive ownership of execution state).
- A step timeout must advance the workflow to the failure path, not leave it stuck in a pending state.

## Known Failure Modes

- **Orphaned workflow**: A crash occurs after step execution but before state persistence — on recovery, the step is re-executed, potentially duplicating side effects.
- **Partial compensation**: A compensating action fails, leaving the system in a partially rolled-back state that requires manual cleanup.
- **Deadlock**: Two workflows hold locks on each other's resources and cannot make progress.
- **Zombie step**: A step timeout fires, the workflow moves to the failure path, but the original step eventually completes and updates the now-stale execution state.
- **State explosion**: The execution state grows with each step (accumulated data, retry counts, timestamps) until it exceeds storage capacity.
- **Infinite retry**: A step persistently fails but the retry policy never marks it as a terminal failure.
- **Cascading failure**: The failure of one workflow triggers compensating actions that fail, triggering more compensating actions in a failure cascade.

## Forces

- **At-most-once vs at-least-once delivery**: At-most-once avoids duplicates but may lose work; at-least-once guarantees progress but requires idempotent handlers.
- **Forward recovery vs backward recovery**: Forward recovery retries failed steps (try again); backward recovery runs compensating actions (undo what was done).
- **Eager vs lazy compensation**: Eager compensation runs as soon as a step fails (minimises locked resources); lazy compensation runs at the end (maximises chance of forward recovery).
- **State size vs recovery speed**: Storing the full accumulated state after every step makes recovery instant but uses storage; storing only the step ID and recomputing on recovery saves space but takes time.
- **Orchestration vs choreography**: Centralised orchestration (engine coordinates all steps) is simpler to manage but a single point of failure; choreography (each step triggers the next) is more distributed but harder to reason about.

## Related Subproblems

- **mars/persist**: Execution state must be durably persisted for crash recovery.
- **saturn/route**: The engine routes the workflow to the correct step handler based on current state.
- **phoebe/resource-lifecycle**: Each workflow execution has a lifecycle (init → step → step → ... → complete/fail).
- **pan/resource-scheduling**: Step timeouts and retry scheduling are resource scheduling problems.
- **jupiter/validate**: Step inputs and outputs must be validated before transitioning.
- **neptune/transform**: Step handlers often transform data from one representation to another.

### ==== Practical Use ====

#### When this pattern matters
Any multi-step process that must survive crashes and partial failures — payment processing, order fulfilment, deployment orchestration, data pipeline orchestration, approval workflows.

#### Start with (minimal form)
A single-threaded loop that reads the current step from a file, executes the step handler, writes the next step to the file, and repeats. No compensation, no parallel execution, no timeout.

```go
func Run(steps []Step) {
    state := readState("workflow.state")
    for i := state.CurrentStep; i < len(steps); i++ {
        result := steps[i].Execute()
        if result.Error != nil {
            return result.Error
        }
        state.CurrentStep = i + 1
        writeState("workflow.state", state)
    }
}
```

#### Split into its own module when (observable triggers)
- A step needs to call an external service that may fail or time out.
- A step must be retried with backoff if it fails.
- A crash during step execution causes duplicated work or lost progress.
- Steps need compensating actions that run in reverse order on failure.
- Multiple workflows run concurrently and must not interfere.

#### Signs you're in this pattern
- You have a multi-step business process that must survive server restarts.
- Each step may call external services that can fail or hang.
- You need to track "which step was last completed" to resume after a crash.
- A failure in a later step requires undoing earlier steps (compensating actions).

#### Real project examples
- **Flowstate** (`engine.go`, `recovery.go`): The canonical instantiation. Workflow steps are executed via the engine, state is persisted atomically in the driver, recovery scans incomplete workflows on startup and resumes from the last committed step.
- **DaemonDB** (WAL checkpoint recovery): Database recovery replays the WAL from the last checkpoint — a simpler form of workflow: `BEGIN → WAL entries × N → CHECKPOINT → switch to new WAL → truncate old`.

#### Common over-engineering traps
- Implementing a full saga orchestration engine before confirming that a single linear retry loop is insufficient.
- Adding distributed locking before workflows run in a single process.
- Building a graphical workflow editor before the step graph is stable.
- Implementing parallel step execution before sequential execution is proven too slow.

## Frame Concern

**Progress across failure boundaries.** Each step may succeed, fail, or time out independently. The system must ensure that the workflow makes progress toward completion despite partial failures — without getting stuck, without duplicating work, and without leaving the system in an inconsistent state. The concern is not individual step reliability but the trajectory of the workflow across many independent failure points.
