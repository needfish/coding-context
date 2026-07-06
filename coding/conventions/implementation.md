# Implementation Process

## Wireframe First

Sketch the user interface layout, screen flow, and interaction model before writing code. Define inputs, outputs, transitions, and error states. Present for review before implementation.

## Stubs and Skeleton

Write a function signature and stub with an entry point. Run it. Inside the top function, call non-existent helpers. Stub each before implementing. Run after each stub. Present for review.

Pick the helper with the fewest parameters and no dependencies on other helpers. Implement with a literal return first. Replace the literal with real logic using the pattern's minimal form. Test and repeat.

For optional fields, choose the absent representation during stubbing -- see [modeling/optionality.md](../modeling/optionality.md).

## Minimal Forms from Patterns

Open each pattern's "Start with (minimal form)" section from [patterns/](../patterns/).

## Data Structure Choice

Start with list/dict/set. When an operation becomes slow or awkward, add an index or change representation. Show the trade-off before refactoring.

## Extract Recurrent Forms

Find recurrent forms (dashboard-stat, page-index). Turn them into explicit types. Move related logic into dedicated functions.

## Section Headers

Add hierarchical section headers. Order top-down (caller before callee). Group constants at top.

## Implementation Guidelines

1. STDLIB FIRST -- does the standard library cover this directly?
2. NATIVE OVER LIBRARIES -- does a built-in language feature or platform API cover this without an import?
3. ONE LINE WHERE POSSIBLE -- can this be expressed as a single expression?
4. EXISTING DEPS ONLY -- is there an already installed dependency that covers this? If not, ask the user.

## Loop Invariants and Higher-Order Functions

See [correctness-reasoning.md](correctness-reasoning.md).
