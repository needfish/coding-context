# jupiter: Validate

> **Pressure class:** Rules variation — rules combine, grow, and change independently.

## Problem

Input data must be checked against a set of rules before it is accepted for processing, ensuring that downstream operations receive data that meets their expectations.

## Context

Arises at every boundary where external data enters the system — user input, API requests, file imports, network messages, inter-service calls. Validation is a gate; it does not transform data, it either passes it through or rejects it with a reason. It is rarely a terminal subproblem — valid data feeds transform (neptune), persist (mars), or route (saturn).

## Domains Involved

- **Input** (lexical): the raw data to be validated. May be structured (JSON, XML, protocol buffer) or unstructured (text, bytes).
- **Rule set** (lexical): the constraints that the input must satisfy. May be a schema, a set of predicates, a formal specification.
- **Result** (lexical): the validated data (if accepted) or a rejection report (if rejected).

## Shared Phenomena

- Input → Validate: raw data, metadata (format hints, provenance)
- Rule set → Validate: constraints, type definitions, allowed ranges, required fields
- Validate → Result: accepted data (typed, coerced) or rejection (field-level errors, codes)

## Invariants

- Accepted data must satisfy every rule in the rule set.
- Rejected data must produce a report that identifies which rule failed and on which field.
- Validation must be deterministic — the same input and rule set must produce the same result every time.
- Validation must not mutate the input or any system state (it is a pure check).

## Known Failure Modes

- **Coercion without confirmation**: The validator silently coerces valid-looking data (e.g., string "123" to integer 123) and the downstream receives a type the author did not intend.
- **Rule contradiction**: Two rules cannot both be satisfied, so no input can ever pass.
- **Over-permissiveness**: A rule is too broad and allows invalid data through.
- **Over-rejection**: A rule is too narrow and rejects valid data.
- **Performance**: Validation rules that query external systems or compute expensive predicates block the caller.
- **Error location loss**: The validator reports "invalid" but does not identify which field or rule failed.
- **Rule set version drift**: The rule set evolves but old input was validated against an older version.

## Forces

- **Strictness vs. tolerance**: Strict validation catches errors early but rejects borderline data that may be acceptable; tolerant validation accepts more but shifts error detection downstream.
- **Early vs. deferred reporting**: Reporting all errors at once helps the caller fix everything; reporting the first error only avoids wasted computation.
- **Schema-first vs. code-first**: Declarative rules (schemas) are easier to audit but less expressive; imperative rules (code) are more flexible but harder to verify completeness.
- **Parsing vs. validation**: Distinguishing structural parsing (is this valid JSON?) from semantic validation (is this value in range?) clarifies responsibility.

## Related Subproblems

- **neptune/transform**: Validation often precedes transformation — validate first, then transform the clean data.
- **mars/persist**: Data is validated before writing.
- **saturn/route**: Incoming requests are validated before routing.
- **mercury/display**: User input through a display requires validation before action.
- **venus/filter**: A filter predicate may need validation.

### ==== Practical Use ====

#### When this pattern matters
Any project that accepts external input — API requests, file imports, user forms, configuration files, network messages.

#### Start with (minimal form)
Inline `if` checks at the point of use. Python: `if not isinstance(value, int): raise ValueError(...)`. Rust: `if value > MAX { return Err(...) }`. No schema library, no validator trait.

#### Split into its own module when (observable triggers)
- The same validation rules are needed in multiple places (duplication emerges).
- The rule set grows beyond what fits on one screen (10+ rules).
- Rules need to be composed (AND/OR across fields) or conditional (field A requires field B).
- Validation needs to produce structured error reports (field-level, code-level).

#### Signs you're in this pattern
- Your code has guard clauses at the start of a function that check input properties and return errors.
- The same validation checks appear in multiple functions with slight variations.
- You need to reject bad input but the checks are interleaved with processing logic.
- Users submit data that causes crashes because boundary conditions were not checked.

#### Real project examples
- **chibicc** (semantic analysis): Type-checking validates expressions against declared types — mismatches produce source-located error messages.
- **Flowstate** (input validation): Workflow step inputs are validated before execution — schema checks prevent downstream corruption.
- **MiniJinja** (`environment.rs` → template compilation): Template syntax validation catches malformed Jinja2 syntax during compilation, not at render time.
- **DaemonDB** (SQL parser): Syntax validation during parsing rejects invalid SQL before execution — early failure prevents wasted work.

#### Common over-engineering traps
- Adopting a schema validation library (JSON Schema, Zod, Pydantic) before you have 5+ fields to validate.
- Building a declarative rule engine before imperative `if` checks are proven to be the bottleneck.
- Validating data that comes from a trusted internal source (defensive programming without the threat model).

## Frame Concern

**Rule set completeness.** The validator can only enforce the rules it is given. It cannot detect that a missing rule would have caught a problem. The frame concern is not how to check rules but how to know enough rules have been written — what the rule set does not say is as important as what it says.
