# ------------------------------------------------------------------------------
#  charon-durable-workflow.md
#  Behavior variation -- the workflow step sequence changes per business process.
# ------------------------------------------------------------------------------

# - Problem -

Your system needs to run a multi-step process that may take hours or days, span multiple services, and survive crashes along the way. Each step must execute at least once, the overall process must eventually finish (success or failure), and failed steps need compensating actions (rollbacks).

# - When You'll See This -

Any system with long-running business processes that span multiple operations, services, or time boundaries. Order fulfilment, payment processing, data migration pipelines, deployment rollouts, document approval chains, and multi-step ETL jobs. The process is broken into discrete steps, each with well-defined input and output.

# - How It Works -

- **Workflow definition**: the static description of the process -- a graph of steps with transitions, retry policies, timeouts, and compensating actions.
- **Workflow instance**: a single running copy of a workflow definition, with its own state (which step it's on, what data it has accumulated).
- **Step**: one unit of work in the workflow -- calling an API, sending an email, writing to a database.
- **Compensating action**: the undo for a step -- if step 3 fails, run the compensation for step 2 to roll back its effects.
- **Workflow state**: persisted progress so that a crash or restart picks up where it left off.

# - What Must Always Be True -

- Each step executes exactly once (or at least once with idempotent retry).
- The workflow state persists between steps so a crash resumes from the last completed step.
- If a step fails, compensating actions run for all previously completed steps (saga pattern).
- The workflow eventually reaches a terminal state (success or failure) without manual intervention.
- Concurrent workflows don't interfere with each other.

# - What Goes Wrong -

- **Step runs twice**: a step executes, the system crashes before persisting completion, and on restart the step runs again -- producing duplicate payments, emails, or API calls.
- **Compensation fails**: a compensating action also fails, leaving the system in an inconsistent state that requires manual repair.
- **Orphaned workflow**: a workflow gets stuck mid-execution and no mechanism detects or recovers it.
- **Timeout misconfiguration**: a step's timeout is too short, causing the workflow to abort a slow but correct operation.
- **State explosion**: storing the full workflow state (including all accumulated data) for thousands of concurrent instances consumes too much storage.
- **Non-deterministic recovery**: the workflow definition changed between the crash and recovery, so the resumed instance runs different logic than the original.

# - Design Trade-Offs -

- **Orchestration vs. choreography**: a central orchestrator manages the workflow (simple to reason about, single point of failure); choreography lets services coordinate via events (distributed, but harder to trace).
- **Eager vs. lazy compensation**: eager compensation runs rollbacks immediately (fast recovery); lazy compensation defers them (may leave resources locked longer).
- **Checkpoint frequency**: checkpoint after every step (more durable, more writes) vs. checkpoint periodically (fewer writes, more work lost on crash).

# - Related Problems -

- **mars/persistence**: workflow state is persisted between steps.
- **saturn/routing**: steps may need routing to different handlers.
- **io/command-execution**: steps may execute external commands.
- **jupiter/validation**: workflow inputs need validation.
- **eris/pipeline**: a workflow is a pipeline that spans time and failures.

# - Getting Started -

A linear sequence of steps in a single script. Each step writes its completion status to a file. On restart, read the file and resume from the last uncompleted step. No orchestration, no compensation, no concurrency.

# - When to Split Into Its Own Module -

- Steps need individual retry policies and timeouts.
- A step failure requires rolling back earlier steps (saga pattern).
- Multiple workflows with different step sequences exist.
- Workflows run concurrently and must not interfere.
- Workflows take hours or days and must survive process restarts.

# - Signs You're in This Pattern -

- Your process has multiple steps that must happen in order.
- A failure in step 3 requires undoing steps 1 and 2.
- The process takes longer than a single request/response cycle.
- Restarting the process from the beginning on failure is too costly.
- You need to track progress across process restarts.

# - Common Mistakes -

- Implementing durable workflows without idempotent steps (a step running twice causes duplicate side effects).
- Not testing crash recovery -- assuming recovery works without actually simulating crashes.
- Building a workflow engine before you need more than two workflows.
- Making compensating actions optional instead of mandatory for every step that has side effects.

# - The Core Challenge -

Durable workflows span time, failures, and service boundaries. Each step may succeed, fail, or hang. The system must persist progress, detect failures, retry transient errors, and compensate for permanent failures -- all while the workflow definition may evolve independently.
