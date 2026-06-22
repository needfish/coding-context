# ------------------------------------------------------------------------------
#  operation-invariants.md
#  Declaring what holds before, during, and after each operation.
# ------------------------------------------------------------------------------

Every operation on a data structure or stateful system has invariants:
conditions that must hold before it runs (preconditions), while it runs
(failure semantics), and after it completes (postconditions). Declaring
these during modeling prevents a class of bugs that testing alone is
poor at catching.

## The Checklist

For each operation (create, query, scan, remove, update, or any
domain-specific verb), answer:

### Empty

What happens when the collection is empty?

- Returns the identity element (0 for sum, `""` for concat, `-inf` for
  max, `true` for AND, `false` for OR).
- Returns an error ("cannot compute max of empty list").
- Is a no-op (removing from an empty set does nothing).
- Returns a default value defined by the caller.

**Identity element**: the value that, when combined with any input via the
operation, leaves it unchanged. Identifying this forces you to decide
the empty case explicitly rather than letting it be a runtime crash.

| Operation | Identity element |
|---|---|
| Sum / addition | `0` |
| Product / multiplication | `1` |
| String concatenation | `""` |
| List concatenation | `[]` |
| Max | `-inf` (or first element) |
| Min | `+inf` (or first element) |
| Boolean AND | `true` |
| Boolean OR | `false` |
| Set union | `{}` (empty set) |
| Set intersection | universal set (first set) |

### Absent

What happens when a key, index, or reference does not exist?

- Returns `null` / `None` / `undefined`.
- Raises an exception / returns an error.
- Creates a default value (insert-on-read).
- Returns a user-supplied fallback.

The choice determines caller obligations. `None` forces the caller to
handle absence. A thrown error forces a try/catch. A created default
can mask bugs. See [optionality.md](optionality.md).

### Boundary

What happens at the edges of the valid range?

- Size 0: empty collection (see "Empty" above).
- Size 1: single-element collection -- does the operation degrade
  (linear scan vs. hash lookup)? Does the single element behave as
  both first and last?
- Max capacity: what happens when the structure is full? Error, block,
  evict oldest, grow?
- Min/max values: what happens when a numeric field is zero, negative,
  or overflows?
- Null/missing: what happens when a required field is absent?

### Partial failure

What happens when an operation fails partway through?

- **Atomic**: the entire operation succeeds or the system is restored
  to the state before it started (transaction, rollback).
- **Idempotent**: running the operation twice produces the same result
  as running it once. The system may be left in a partially-applied
  state, but re-applying completes it.
- **Best-effort**: partial results are visible even if the operation
  did not complete. Callers must handle intermediate states.

## Example: Key-Value Store

| Property | Invariant |
|---|---|
| Empty | Querying an empty store returns `None`. Scanning returns `[]`. |
| Absent | `get(missing_key)` returns `None`. `delete(missing_key)` is a no-op. |
| Boundary | At size 0, `len == 0`. At max capacity, `put` returns `StoreFull` error. |
| Partial failure | `put` is atomic -- a crash during write leaves the old value intact. |

## Example: Sort Operation

| Property | Invariant |
|---|---|
| Empty | Sorting `[]` returns `[]`. |
| Absent | N/A (no keys, no references). |
| Boundary | Sorting `[x]` returns `[x]`. Sorting a collection where all elements compare equal returns the collection in its original order (stable sort). |
| Partial failure | Sorting has no partial failure -- it produces a fully-sorted result or does not run. |

## Common Mistakes

- **Assuming non-empty**: writing `max = list[0]` without checking for
  empty, then crashing on empty input.
- **Ignoring partial failure**: assuming an operation either succeeds or
  fails cleanly, when in practice a crash mid-write can leave corrupted
  state.
- **Forgetting the identity element**: using `0` as the initial value for
  a product operation, making every result `0`.
- **Mixing absent and empty**: treating a missing key the same as an
  empty collection, when the domain distinguishes "no list" from "empty
  list."

## Invariants

- Every operation must declare its behavior on empty input and absent
  references -- silence is a bug.
- The identity element for any accumulation operation must be explicit.
- Partial failure semantics (atomic / idempotent / best-effort) must be
  documented for every operation that spans multiple steps.
- Boundary values (size 0, size 1, max capacity) must be tested for
  every operation -- they are where most bugs live.

## See Also

- [optionality.md](optionality.md) -- modeling absent values (Option vs.
  null vs. sentinel vs. empty collection)
- [type-precision.md](type-precision.md) -- choosing precise types so
  invalid states are unrepresentable
- [testing/testing.md](../testing/testing.md) -- the Data Representation
  Walkthrough mirrors this checklist for the Evaluate phase
- [patterns/janus-reduction-aggregation.md](../patterns/janus-reduction-aggregation.md) --
  identity element for empty collections in reductions
