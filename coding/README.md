# coding

Subdirectories: toolchain/, refactoring/, modeling/, conventions/, heuristics/, error-handling/, testing/, project-conventions/, patterns/.

## Interaction Rules

- Give direct answers. Skip preamble and postamble.
- For scaffolding steps, say only "Please review" or "Ready for next step."
- On confirmation, recap the next step in one sentence then execute it.
- When asked to do something, execute the task directly.
- When reasoning reaches an assumption, ask the user.
- Write content using positive grammar (A does B and avoids C).

## Development Constraints

- Open the catalog at [~/.context/coding/](~/.context/coding/) when starting a new project or refactoring. Start with [how-to-use.md](~/.context/coding/how-to-use.md) to find project type and core patterns.
- Serve only the immediate request. No speculative extras.
- Use the standard library when it covers the need.
- Prefer built-in platform features.
- Use an already-installed dependency when it covers the request. Ask before adding new ones.
- Solve new problems with the shortest correct code.
- Comments reference docs links when the function or library use case is not the most common one.

## Defaults

| Domain | Reference |
|--------|-----------|
| Web stack | [toolchain/web-stack.md](toolchain/web-stack.md) (default Level 2: Preact + HTMX) |
| Toolchain | [toolchain/toolchain-defaults.md](toolchain/toolchain-defaults.md) (package managers, per-language tooling, pre-commit, EditorConfig) |
| Linting/SAST | [toolchain/safety-tools.md](toolchain/safety-tools.md) |
| Invariants | [modeling/invariants.md](modeling/invariants.md) (categories and encoding strategy) |
| Temp files | Clean up tmp/scratch files after the task is done unless the user says to keep them |
| Utility scripts | Add conversions, analysis, data processing scripts to a `scripts/` directory at project root |

## Workflow

Each design problem cycles through Model -> Build -> Evaluate -> Reflect.

| Phase | Reference |
|-------|-----------|
| 1: Model | [modeling/process.md](modeling/process.md) (decomposition, types, entity relationships, invariants) |
| 2: Build | [conventions/implementation.md](conventions/implementation.md) (wireframing, stubs, minimal forms, guidelines) |
| 3: Evaluate | [testing/testing.md](testing/testing.md) (edge cases, data walkthrough, property-based testing) |
| 4: Reflect | [refactoring/reflect.md](refactoring/reflect.md) (assumptions, splitting, git analysis, 5 whys) |

## Heuristics

See [heuristics/heuristics.md](heuristics/heuristics.md).

## Modeling Communication

See [modeling/modeling-communication.md](modeling/modeling-communication.md).

## Conventions

| Convention | Reference |
|------------|-----------|
| Naming | [conventions/naming.md](conventions/naming.md) (style, grammar patterns, types, functions, variables, vocabulary) |
| Layout | [conventions/layout.md](conventions/layout.md) (heading format, file naming) |
| Comments | [conventions/comments.md](conventions/comments.md) |
| Error handling | [error-handling/error-handling.md](error-handling/error-handling.md) |
| Testing | [testing/testing.md](testing/testing.md) |
| Project conventions (git, dependencies, README) | [project-conventions/project-conventions.md](project-conventions/project-conventions.md) |
