# Modeling Process

## Problem Statement

Write a one-sentence problem statement extracted from the user's request.

## Decompose Using the Pattern Catalog

Find the project type in [how-to-use.md](../how-to-use.md) -> project type table -> core patterns. Open each core pattern from [patterns/](../patterns/) and extract its Domain, Invariants, Related Subproblems, and Decomposition sections.

## Define Types and Operations

Define input schema (product types), output schema (product types), state transitions (sum types / discriminated unions), operation signatures (verb phrases), functional dependencies, and invariants from what the system does, not how.

| File | Guidance |
|---|---|
| [type-precision.md](type-precision.md) | Prefer ADTs over primitives. Avoid stringly-typed fields -- use literal unions, enums, branded types, or value objects. |
| [optionality.md](optionality.md) | Choose between Option/Maybe, null, empty collection, sentinel, or separate record types. |
| [state-machine.md](state-machine.md) | Model transitions as sum types so illegal states are unrepresentable. |

## Entity Relationships

Open [many-forms.md](many-forms.md) to compare alternative representations before committing to one. Name the relationship from the domain. Distinguish entities (things with identity) from values (replaceable attributes). Distinguish cardinality (1:1, 1:N, M:N). If the same data appears in multiple relationships, extract it as its own relation and reference it.

## Invariants

See [invariants.md](invariants.md) for categories.

## Data Representation Decomposition

If the problem is a data representation choice, list the operations (create, query, scan, remove, update, atomicity requirements) and propose a decomposition: one container per access path, join at shared root when they reach the same leaf, sharing requires an FD. Fusion merges leaf objects; cross-linking adds shortcuts. Start with 2-3 edges; more than 5 suggests the relation should be split.

For each operation, declare its invariants: behavior on empty collection, behavior on absent key, behavior at boundaries (size 0, 1, max), and partial failure semantics. See [operation-invariants.md](operation-invariants.md).
