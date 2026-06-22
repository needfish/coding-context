# Implementation Process

## Wireframe First

Sketch the user interface layout, screen flow, and interaction model before writing code. Use low-fidelity wireframes (boxes, labels, arrows) to map each user goal to a screen sequence. Define inputs, outputs, transitions, and error states. Present for review before implementation.

## Stubs and Skeleton

Write a function signature and stub with an entry point. Run it (does nothing). Show the running skeleton. Inside the top function, call non-existent helpers. Stub each before implementing. Run after each stub. Present for review.

Pick the helper with the fewest parameters and no dependencies on other helpers. Implement with a literal return (empty list, null, zero) first. Replace the literal with real logic using the pattern's minimal form. Test and repeat.

## Minimal Forms from Patterns

Open each pattern's "Start with (minimal form)" section from [patterns/](../patterns/). These give the simplest correct implementation for each subproblem.

## Data Structure Choice

Start with list/dict/set. When an operation becomes slow or awkward, refactor by adding an index or changing representation. Show the trade-off before refactoring.

## Extract Recurrent Forms

Find recurrent forms (dashboard-stat, page-index). Turn them into explicit types. Move related logic into dedicated functions.

## Section Headers

Add hierarchical section headers. Order top-down (caller before callee). Group constants at top.

## Implementation Guidelines

1. **STDLIB FIRST** --- does the standard library have a function/class that directly solves this?
2. **NATIVE OVER LIBRARIES** --- does a built-in language feature or platform API cover this without an import?
3. **ONE LINE WHERE POSSIBLE** --- can this be expressed as a single expression?
4. **EXISTING DEPS ONLY** --- is there already an installed dependency that covers this? If not, ask the user.
