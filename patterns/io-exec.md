# io: Command Execution

> **Pressure class:** Object variation — the external command interface changes while the execute-per-item contract stays the same.

## Problem

A collection of items must be processed by an external command, with each item substituted into the command template, potentially running in parallel or as a batch.

## Context

Arises whenever a tool finds results and the user wants to act on them — unzipping archives, opening files in an editor, converting image formats, running analyses. The command execution subproblem sits after a search/collection subproblem (callisto/walk, venus/filter) and before result collection or display.

## Domains Involved

- **Items** (lexical): the collection of paths or values to be processed.
- **Command template** (lexical): the external command with placeholder tokens (`{}`, `{.}`, `{/}`, etc.) that are replaced with item values.
- **Process** (causal): the OS-level process spawned for each execution.
- **Results** (lexical): the exit statuses and output of each command invocation.

## Shared Phenomena

- Items → Executor: paths, values
- Command template → Executor: template string, placeholder definitions
- Executor → Process: command line (after substitution), working directory
- Process → Executor: exit code, stdout, stderr
- Executor → Results: aggregated exit codes, per-item status

## Invariants

- Each item must be substituted into the command template exactly as specified by the placeholder rules.
- The command must be executed for every item, unless a prior invocation fails and the execution mode requires early abort.
- Parallel executions must not interfere with each other — each process has isolated stdin/stdout/stderr.
- The executor must not deadlock when the number of items exceeds the parallelism limit and subprocesses produce output that fills system buffers.

## Known Failure Modes

- **Command injection**: A malicious item value (e.g., a filename containing `; rm -rf /`) is substituted literally into a shell command, causing unintended execution.
- **Argument overflow (batch mode)**: Too many items are passed as arguments, exceeding the OS maximum argument length (`ARG_MAX`).
- **Resource exhaustion**: Too many parallel processes exhaust system memory, file descriptors, or process table entries.
- **Output interleaving**: Multiple parallel processes write to stdout simultaneously, producing garbled interleaved output.
- **Deadlock on output buffers**: A subprocess blocks writing to stdout because the parent is not reading, while the parent waits for the subprocess to exit.
- **Placeholder mismatch**: The command template contains a placeholder token that the executor does not recognise, leading to literal (unsubstituted) characters in the command.

## Forces

- **Per-item vs. batch**: Per-item execution (`-x`) spawns one process per item (simple, parallelisable); batch execution (`-X`) passes all items to one process (efficient for tools that accept many arguments, but risks ARG_MAX).
- **Sequential vs. parallel**: Sequential execution is predictable and resource-friendly; parallel execution is faster but introduces resource contention and nondeterministic output order.
- **Shell vs. direct exec**: Executing through a shell (`sh -c`) supports pipes, redirects, and environment variables but introduces injection risk; direct `execv` is safe but limited to simple commands.
- **Synchronous vs. detached**: Waiting for each process to finish gives deterministic progress; detaching processes (fire-and-forget) is faster but loses error information.

## Related Subproblems

- **venus/filter**: Items are often filtered before execution.
- **neptune/transform**: Item values may need transformation before substitution (extension removal, path relativisation).
- **jupiter/validate**: The command template must be validated before execution.
- **mercury/display**: Execution progress and results are typically displayed.
- **callisto/walk**: Walk results are often the items passed to execution.

### ==== Practical Use ====

#### When this pattern matters
Any project that runs external commands on found items — batch processing, build tools, file conversion, automated analysis.

#### Start with (minimal form)
`subprocess.run()` (Python), `std::process::Command` (Rust), `child_process.execSync()` (Node). Run sequentially, collect output, print results. No parallelism, no placeholder substitution.

#### Split into its own module when (observable triggers)
- The command template needs placeholder substitution (`{}`, `{.}`, `{/}`).
- Items must be processed in parallel (thread pool, not just sequential loop).
- Batch execution (passing all items as args) risks `ARG_MAX` overflow.
- You need to handle subprocess failure modes (timeout, output buffer deadlock, exit code aggregation).

#### Signs you're in this pattern
- Your code calls external programs using `subprocess`, `os.exec`, `Command`, or `Process`.
- You need to process a collection of items, where each item requires executing an external command.
- Error handling around subprocess calls is repetitive (check exit code, capture stderr).
- You're building pipelines of commands where the output of one feeds the input of another.

#### Real project examples
- **Helix editor** (language server integration): Spawns LSP processes, communicates via stdin/stdout JSON-RPC — each LSP server is a long-running subprocess.
- **yarsync** (rsync delegation): File synchronisation delegates to rsync for the actual transfer — yarsync manages the command-line invocation and output parsing.
- **pnpm** (package scripts): Executes lifecycle scripts (install, build, test) as subprocesses, collecting output and exit codes per script.

#### Common over-engineering traps
- Building a thread pool before measuring that sequential execution is too slow.
- Abstracting the executor into a generic pipeline before you have a second pipeline stage.
- Supporting shell features (pipes, redirects) before direct exec proves insufficient.

## Frame Concern

**Subprocess lifecycle vs. consumer expectations.** The subprocess runs in a separate address space with its own I/O buffers, environment, and lifetime. The executor must coordinate when to spawn, when to read output, when to wait, and what to do when a subprocess fails — without knowing how long the subprocess will run or how much output it will produce.
