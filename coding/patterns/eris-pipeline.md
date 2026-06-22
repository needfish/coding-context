# ------------------------------------------------------------------------------
#  eris-pipeline.md
#  Behavior variation -- the processing algorithm at each stage changes independently.
# ------------------------------------------------------------------------------

# - Problem -

A single input must go through a fixed sequence of processing stages, where each stage consumes the output of the previous one. Errors from early stages should prevent later stages from running, and shared context must be available to all stages.

# - When You'll See This -

Whenever a complex transformation can be broken into a linear sequence of simpler steps. The classic example is a compiler: tokenize -> preprocess -> parse -> type-check -> codegen. Each stage works on a different representation (characters -> tokens -> AST -> typed AST -> assembly). Found in compilers, ETL pipelines, image processing chains, build systems, and multi-step validators.

# - How It Works -

- **Input**: the initial data at the start of the pipeline.
- **Stages**: a fixed sequence of processing steps, each taking input and producing output for the next stage.
- **Shared context**: information accessible to all stages -- a symbol table, configuration, source location mapping.
- **Error collector**: accumulates errors from each stage. Early-stage errors should short-circuit later stages.
- **Output**: the final result after all stages have processed the input.

# - What Must Always Be True -

- Stages execute in order -- no stage runs before its dependencies.
- Each stage receives the output of the previous stage (or a sensible default if the previous stage was skipped).
- An error in an early stage prevents later stages from running (no wasted work on bad input).
- Shared context is read-only after initialisation (stages don't overwrite each other's data).
- Each stage is independently testable.

# - What Goes Wrong -

- **Error recovery ambiguity**: a stage produces partial output despite an error, and the next stage operates on corrupted data.
- **Context coupling**: stages share too much mutable state, making it hard to reason about data flow.
- **Pipeline explosion**: the pipeline grows to 15+ stages, making it hard to understand the overall flow.
- **Duplicate work**: multiple stages independently compute the same derived data because shared context isn't used.
- **Hidden dependencies**: stage B depends on stage A's side effect, not its declared output, so reordering breaks things silently.

# - Design Trade-Offs -

- **Fail-fast vs. collect-all**: fail-fast stops at the first error (fast, clear); collect-all runs all stages and reports all errors (complete, but wastes work on bad input).
- **Sequential vs. parallel stages**: most pipelines are sequential, but some stages can run in parallel if they only depend on shared context, not on each other's output.
- **Rigid vs. dynamic pipelines**: a fixed stage order is predictable and testable; a dynamic pipeline that adapts to input is flexible but harder to reason about.

# - Related Problems -

- **neptune/transformation**: each pipeline stage is a transformation.
- **jupiter/validation**: validation stages gate the pipeline.
- **pluto/input-parsing**: the first stage is often parsing.

# - Getting Started -

Call functions in sequence, passing the result of each to the next. Python: `result = stage3(stage2(stage1(input)))`. If any stage returns an error, skip the remaining stages.

# - When to Split Into Its Own Module -

- The pipeline grows beyond 5 stages.
- Stages need independent testing.
- You need to share context (symbol table, config, metrics) across stages.
- Some stages are optional or conditional based on the input.

# - Signs You're in This Pattern -

- Your code calls functions in a fixed sequence, each one transforming the output of the previous.
- A compilation, build, or data processing step has multiple phases.
- An error in an early phase produces confusing errors in later phases.
- You need to thread shared state through a chain of operations.

# - Common Mistakes -

- Making stages depend on each other's side effects instead of declared outputs.
- Building a pipeline framework before you need more than 3 stages.
- Letting the pipeline grow without extracting reusable stage abstractions.
- Not short-circuiting on errors -- processing bad input through all stages.

# - The Core Challenge -

A pipeline decomposes a complex process into independent stages. The challenge is keeping stages truly independent -- each should depend only on the previous stage's output and the shared context, not on hidden state, side effects, or implicit ordering assumptions.
