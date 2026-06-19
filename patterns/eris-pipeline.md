# eris: Multi-Stage Pipeline

> **Pressure class:** Behavior variation  --  the processing algorithm at each stage changes independently.

## Problem

A single input must be processed through a fixed sequence of transformation stages, where each stage consumes the output of the previous one, error information must propagate backward to the user, and shared context must be threaded through every stage.

## Context

Arises whenever a complex transformation is decomposed into a linear sequence of simpler steps. The canonical example is the compiler pipeline: tokenize  ->  preprocess  ->  parse  ->  type-check  ->  codegen. Each stage operates on a different representation (characters  ->  tokens  ->  AST  ->  typed AST  ->  assembly), and early-stage errors should prevent later-stage work. The shared context (symbol table, configuration, source location mapping) must be available to all stages. Found in compilers, ETL pipelines, image processing chains, build systems, and multi-step validators.

## Domains Involved

- **Input** (lexical): the initial representation at the start of the pipeline.
- **Stages** (causal): a fixed sequence of processing functions. Each stage reads from the previous stage's output representation and writes to the next stage's input representation.
- **Shared context** (lexical): information that every stage needs access to  --  configuration, symbol tables, source location mapping, error aggregator.
- **Error sink** (lexical): accumulates errors from all stages, typically with source location information. Early errors may cause later stages to produce degraded output or skip processing entirely.

## Shared Phenomena

- Input  ->  Stage 1: initial representation
- Stage N  ->  Stage N+1: transformed representation (AST, tokens, etc.)
- Shared context  ->  All stages: config, symbol table, source locations
- Stage N  ->  Error sink: error/warning with location metadata
- Error sink  ->  Pipeline controller: error count, severity summary
- Pipeline controller  ->  Stages: abort signal (if error threshold exceeded)

## Invariants

- The stage order is fixed  --  output of stage N is input to stage N+1.
- An error in stage N must not corrupt the shared context used by earlier stages (already processed) or later stages (not yet run).
- The pipeline must make progress: if no errors occur, every stage must eventually process the input.
- Errors must be accumulated, not truncated  --  the user should see all errors from all stages in a single report.
- Each stage must handle gracefully the case where the previous stage produced degraded output (e.g., parsing succeeded with errors  --  codegen must still produce something reasonable).
- The pipeline must be restartable  --  running the pipeline twice on the same input must produce the same result (deterministic stages).

## Known Failure Modes

- **Error cascade**: An early error causes downstream stages to produce confusing secondary errors, burying the root cause.
- **Context poisoning**: An early stage modifies shared context in a way that breaks later stages, even though no error was reported.
- **Degenerate output**: A stage receives degraded input from a previous stage (due to error recovery) but does not handle it, producing garbage.
- **Orphaned errors**: An error is reported but the pipeline continues processing as if nothing happened, wasting work.
- **Checkpoint inconsistency**: The pipeline is interrupted mid-way and cannot be resumed because stages are not idempotent.
- **Memory blowup**: Intermediate representations between stages are kept in memory until the pipeline completes, causing OOM for large inputs.

## Forces

- **Batch vs. streaming**: Batch pipelines process the entire input through one stage before moving to the next (simple, high memory); streaming pipelines pass data between stages incrementally (complex, low memory).
- **Fail-fast vs. fail-soft**: Fail-fast aborts on the first error (saves work, but user only sees one error at a time); fail-soft collects all errors (max info, but may waste work on degraded input).
- **Sequential vs. parallel stages**: Sequential stages are simple but waste idle cores; parallel stages (pipelining multiple inputs) increase throughput but add synchronisation complexity.
- **Homogeneous vs. heterogeneous stages**: A pipeline of similar stages (filter chain) is easy to parameterise; a pipeline of dissimilar stages (compiler) has unique error handling per stage.

## Related Subproblems

- **neptune/transformation**: Each individual stage is a transform; the pipeline composes transforms.
- **jupiter/validation**: Type-checking and semantic analysis are validation stages in a compiler pipeline.
- **saturn/routing**: The pipeline controller routes the input through stages in order.
- **mars/persistence**: Intermediate pipeline results may be cached to disk for incremental builds.
- **phoebe/resource-lifecycle**: Each pipeline stage may own resources that must be initialised before and cleaned up after the stage runs.
- **io/command-execution**: A build system's compilation pipeline delegates each stage to an external process.

### ==== Practical Use ====

#### When this pattern matters
Any project that processes input through a fixed sequence of distinct representations  --  compilers, transpilers, ETL jobs, image processing chains, multi-phase validators, build systems.

#### Start with (minimal form)
A single function that calls sub-functions in order. No abstraction for the pipeline itself. Error handling: check return value after each call and abort. Shared context: a struct passed by pointer to each function.

```c
int process(char *input) {
    Tokens *tokens = tokenize(input);
    if (!tokens) return -1;
    AST *ast = parse(tokens);
    if (!ast) return -1;
    return codegen(ast, stdout);
}
```

#### Split into its own module when (observable triggers)
- A stage takes long enough that the user needs progress feedback.
- The pipeline needs to support multiple input types or multiple output formats.
- Intermediate representations need caching for incremental processing.
- You need to run multiple inputs through the pipeline concurrently.
- Error recovery becomes complex (warnings vs errors, error count limits, error suppression).

#### Signs you're in this pattern
- You have a sequence of function calls where each call transforms the output of the previous one.
- Error handling repeats the same pattern between each stage (check result  ->  abort or continue).
- The intermediate data types between stages form a chain (type A  ->  type B  ->  type C).
- You can visualise processing as a linear flow diagram with boxes and arrows.

#### Real project examples
- **chibicc** (compiler pipeline): `tokenize  ->  preprocess  ->  parse  ->  codegen`  --  each stage produces a different representation (tokens  ->  AST  ->  assembly), early-stage errors block later stages.
- **MiniJinja** (template compilation): `parse  ->  compile  ->  VM execute`  --  template text  ->  AST  ->  bytecode  ->  rendered output, with error propagation from parse to execute.
- **DaemonDB** (query execution): `SQL parse  ->  AST  ->  bytecode  ->  B-tree traversal`  --  each stage validates the previous stage's output format.
- **lite3** (serialisation pipeline): `object  ->  tagged-binary  ->  B-tree leaf  ->  page write`  --  four distinct representations, shared buffer context.

#### Common over-engineering traps
- Building an abstract "pipeline framework" before you have a second distinct pipeline.
- Implementing parallel streaming when batch processing is fast enough.
- Adding dependency analysis between stages before you have enough stages to need it.

## Frame Concern

**Stage coupling.** Each stage depends on the output representation of the previous stage. Changing a stage's output format may break every downstream stage, but the shared context (symbol tables, configuration, error state) creates implicit coupling that is easy to overlook. The concern is how to keep stages independently changeable when the data flowing between them is tightly coupled to each stage's logic.
