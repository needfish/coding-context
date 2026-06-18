# neptune: Transform

## Problem

Data in one representation must be converted to another representation, changing its structure, encoding, or level of abstraction without changing its essential content.

## Context

Arises whenever two parts of the system use different representations for semantically equivalent data. The boundary between internal and external formats (serialisation), between layers of abstraction (DTO mapping), between system versions (migration). Transform is a connecting subproblem — it sits between two other problem types that would otherwise be incompatible.

## Domains Involved

- **Source representation** (lexical): the input data in its original form.
- **Target representation** (lexical): the output data in the desired form.
- **Mapping** (lexical): the rules that define how source elements correspond to target elements.

## Shared Phenomena

- Source → Transform: data, metadata (format, version)
- Mapping → Transform: field correspondences, type coercions, default values
- Transform → Target: converted data

## Invariants

- The transformation must be reversible if the mapping is bidirectional (round-trip: transform then inverse must yield the original).
- Elements in the source that have no corresponding element in the target must be handled explicitly (dropped, defaulted, or error).
- The transformation must not introduce information not present in the source, unless the mapping explicitly defines a derivation.
- The transformation must not have side effects on the source or external systems.

## Known Failure Modes

- **Information loss**: The target representation cannot express some information present in the source, and the loss goes undetected.
- **Implicit coercion**: The mapping silently converts a value to a type or range that changes its meaning (e.g., float to int truncation).
- **Incomplete mapping**: Some source fields have no target field defined, and the transformer fails or produces incomplete output.
- **Cyclic structure**: The source contains cycles that the transformer does not handle, causing infinite recursion or stack overflow.
- **Version mismatch**: The source was produced by a different version of the schema than the mapping expects.
- **Identity loss**: Two source values that are semantically distinct (e.g., "123" and 123) become identical in the target.

## Forces

- **Explicitness vs. brevity**: An explicit mapping lists every field correspondence (clear but verbose); an implicit mapping uses conventions and defaults (compact but surprising).
- **Strictness vs. flexibility**: A strict transform fails on unexpected input; a flexible transform skips or defaults unknown fields.
- **In-place vs. copy**: Transforming in place avoids allocation but risks aliasing; copying is safe but costs memory and time.
- **Schema-driven vs. code-driven**: Declarative mappings are easier to verify; procedural transformations are more powerful.

## Related Subproblems

- **mars/persist**: Serialisation for storage is a transform.
- **mercury/display**: Data may need transformation before display.
- **jupiter/validate**: Validation often precedes transformation.
- **venus/filter**: Filter may produce a subset that then needs transformation.
- **earth/sort**: Sort keys may need transformation before comparison.

## Frame Concern

**Semantic equivalence across heterogeneous representations.** Two representations rarely align perfectly. The mapping must decide what to do when the source has more information than the target, less information, or different structure. The concern is not the mechanics of conversion but the handling of structural mismatch — every transform is a lossy compression of semantics into a different shape.
