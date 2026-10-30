# Invariants Reference

## Categories of Invariants

| Category | Description |
|---|---|
| Functional dependencies | Which combinations uniquely determine which others |
| Domain constraints | Range (NonNegative), cardinality, nullability, value set (enum) |
| Temporal invariants | start <= end, created <= updated, version monotonic |
| Aggregate invariants | count >= 0, balance in sync, sum of parts equals whole |
| State machine invariants | Valid transitions only (pending -> active -> archived) |
| Referential integrity | Every reference points to an existing entity |
| Encoding invariants | Canonical form (dates UTC, strings normalized) |
| Operation invariants | Behavior on empty collection, absent key, boundary values, and partial failure for each operation |

## Encoding Strategy

Encode invariants at the closest boundary to their domain, in order of preference:

1. **Compile-time types** -- branded types, discriminated unions, literal types, generics. Zero runtime cost. See [type-precision.md](type-precision.md).
2. **Parse boundaries** -- Result-returning constructors. Cost at entry only.
3. **Debug asserts** -- for invariants the type system cannot express. Removed in release builds.
4. **Runtime guards** -- last resort for dynamic data.

Refine types early (sum types, product types, branded types, literal unions). Defer implementation through stubs, interfaces, and internal details. Never defer operation signatures or invariants.

## Expanded Guidance

| File | Covers |
|---|---|
| [type-precision.md](type-precision.md) | Encoding invariants and domain constraints: branded types, newtypes, literal unions, value objects |
| [optionality.md](optionality.md) | Domain constraints: nullability, absent values, Option/Maybe vs. sentinel vs. empty collection |
| [state-machine.md](state-machine.md) | State machine invariants: sum-type states, illegal states unrepresentable, per-state data |
| [operation-invariants.md](operation-invariants.md) | Operation invariants: empty, absent, boundary, partial failure checklist |

## Property-Based Testing

When the domain has clear invariants, define the invariant and let the framework generate counterexamples. Prefer this over hand-writing every edge case. Fall back to example-based testing for purely sequential or IO-bound logic.

## See Also

- [process.md](process.md)
- [testing/testing.md](../testing/testing.md)
