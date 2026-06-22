# Invariants Reference

Invariants are conditions that must always hold true for a system to be correct.
They are the backbone of reliable design.

## Categories of Invariants

Declare these when modeling a system:

- **Functional dependencies** -- which combinations uniquely determine which others?
- **Domain constraints** -- range (NonNegative), cardinality, nullability, value set (enum).
- **Temporal invariants** -- start <= end, created <= updated, version monotonic.
- **Aggregate invariants** -- count >= 0, balance in sync, sum of parts equals whole.
- **State machine invariants** -- valid transitions only (pending -> active -> archived).
- **Referential integrity** -- every reference points to an existing entity.
- **Encoding invariants** -- canonical form (dates UTC, strings normalized).
- **Operation invariants** -- behavior on empty collection, absent key, boundary
  values, and partial failure for each operation.

## Encoding Strategy

Encode invariants at the closest boundary to their domain, in order of preference:

1. **Compile-time types** -- branded types, discriminated unions, literal types,
   generics. Zero runtime cost. See [type-precision.md](type-precision.md).
2. **Parse boundaries** -- `Result`-returning constructors. Cost at entry only.
3. **Debug asserts** -- for invariants the type system cannot express. Removed
   in release builds.
4. **Runtime guards** -- last resort for dynamic data.

Refine types early (sum types, product types, branded types, literal unions).
Defer implementation through stubs, interfaces, and internal details. Never
defer operation signatures or invariants.

## Expanded Guidance

Open these files for expanded treatments of each invariant category:

- [type-precision.md](type-precision.md) -- encoding invariants & domain constraints:
  branded types, newtypes, literal unions, value objects
- [optionality.md](optionality.md) -- domain constraints: nullability, absent values,
  Option/Maybe vs. sentinel vs. empty collection
- [state-machine.md](state-machine.md) -- state machine invariants: sum-type states,
  illegal states unrepresentable, per-state data
- [operation-invariants.md](operation-invariants.md) -- operation invariants:
  empty, absent, boundary, partial failure checklist

## Property-Based Testing

When the domain has clear invariants, define the invariant and let the
framework generate counterexamples. Prefer this over hand-writing every edge
case. Fall back to example-based testing for purely sequential or IO-bound
logic.

## See Also

- [process.md](process.md) -- the Modeling phase workflow
- [testing/testing.md](../testing/testing.md) -- evaluating invariants through
  the Data Representation Walkthrough
