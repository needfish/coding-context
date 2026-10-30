# coding-context

Subdirectories: toolchain/, refactoring/, modeling/, conventions/, heuristics/, error-handling/, testing/, project-conventions/, application-patterns/, system-patterns.

## Interaction Rules

- Give direct answers. Skip preamble and postamble.
- For scaffolding steps, say only "Please review" or "Ready for next step."
- On confirmation, recap the next step in one sentence then execute it.
- When asked to do something, execute the task directly.
- When reasoning reaches an assumption, ask the user.
- Write content using positive grammar (A does B and avoids C).

## Development Constraints

- Open this catalog when starting a new project or refactoring. Start with [how-to-use.md](how-to-use.md) to find project type and core patterns.
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

## File Index

### modeling/
| File | Purpose |
|---|---|
| [process.md](modeling/process.md) | Modeling workflow: decomposition, types, operations, entities |
| [invariants.md](modeling/invariants.md) | Categories of invariants and encoding strategy |
| [type-precision.md](modeling/type-precision.md) | Precise types (ADTs, branded types, newtypes) over primitives |
| [optionality.md](modeling/optionality.md) | Modeling absent/optional values (Option, null, sentinel) |
| [state-machine.md](modeling/state-machine.md) | State transitions as sum types |
| [operation-invariants.md](modeling/operation-invariants.md) | Invariants for each operation (empty, absent, boundary, partial failure) |
| [many-forms.md](modeling/many-forms.md) | Kent's catalog of 37+ representations for a single fact |
| [modeling-communication.md](modeling/modeling-communication.md) | How to communicate modeling decisions |

### conventions/
| File | Purpose |
|---|---|
| [naming.md](conventions/naming.md) | Naming style, grammar patterns, types/functions/variables/constants |
| [layout.md](conventions/layout.md) | Code layout, heading format, file naming |
| [comments.md](conventions/comments.md) | Comment conventions (say why, not what) |
| [implementation.md](conventions/implementation.md) | Implementation workflow (wireframes, stubs, minimal forms) |
| [correctness-reasoning.md](conventions/correctness-reasoning.md) | Loop invariants, higher-order functions, correctness reasoning |

### refactoring/
| File | Purpose |
|---|---|
| [reflect.md](refactoring/reflect.md) | Reflection process: assumptions, splitting triggers, co-change analysis, 5 whys |
| [refactoring-catalog.md](refactoring/refactoring-catalog.md) | Refactoring catalog across OOP, functional, and data-oriented paradigms |
| [defunctionalization.md](refactoring/defunctionalization.md) | Cross-paradigm transformation: higher-order functions to data types and back |

### toolchain/
| File | Purpose |
|---|---|
| [toolchain-defaults.md](toolchain/toolchain-defaults.md) | Package managers, per-language toolchain, EditorConfig, pre-commit hooks |
| [web-stack.md](toolchain/web-stack.md) | Web app stack levels (default: Level 2 -- Preact + HTMX) |
| [safety-tools.md](toolchain/safety-tools.md) | Linting, static analysis, and safety enforcement tools per language |
