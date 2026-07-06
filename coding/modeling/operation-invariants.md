# Operation Invariants

For each operation (create, query, scan, remove, update, or any domain-specific verb), declare:

## Empty

What happens when the collection is empty?

- Returns the identity element (0 for sum, "" for concat, -inf for max, true for AND, false for OR).
- Returns an error ("cannot compute max of empty list").
- Is a no-op (removing from an empty set does nothing).
- Returns a default value defined by the caller.

| Operation | Identity element |
|---|---|
| Sum / addition | 0 |
| Product / multiplication | 1 |
| String concatenation | "" |
| List concatenation | [] |
| Max | -inf (or first element) |
| Min | +inf (or first element) |
| Boolean AND | true |
| Boolean OR | false |
| Set union | {} (empty set) |
| Set intersection | universal set (first set) |

## Absent

What happens when a key, index, or reference does not exist?

- Returns null / None / undefined.
- Raises an exception / returns an error.
- Creates a default value (insert-on-read).
- Returns a user-supplied fallback.

The choice determines caller obligations. None forces the caller to handle absence. A thrown error forces a try/catch. A created default can mask bugs. See [optionality.md](optionality.md).

## Boundary

What happens at the edges of the valid range?

- Size 0: empty collection (see Empty above).
- Size 1: single-element collection -- does the operation degrade (linear scan vs. hash lookup)? Does the single element behave as both first and last?
- Max capacity: what happens when the structure is full? Error, block, evict oldest, grow?
- Min/max values: what happens when a numeric field is zero, negative, or overflows?
- Null/missing: what happens when a required field is absent?

## Partial failure

What happens when an operation fails partway through?

- **Atomic**: the entire operation succeeds or the system is restored to the state before it started (transaction, rollback).
- **Idempotent**: running the operation twice produces the same result as running it once.
- **Best-effort**: partial results are visible even if the operation did not complete. Callers must handle intermediate states.

## Example: Key-Value Store

| Property | Invariant |
|---|---|
| Empty | Querying an empty store returns None. Scanning returns []. |
| Absent | get(missing_key) returns None. delete(missing_key) is a no-op. |
| Boundary | At size 0, len == 0. At max capacity, put returns StoreFull error. |
| Partial failure | put is atomic -- a crash during write leaves the old value intact. |

## Example: Sort Operation

| Property | Invariant |
|---|---|
| Empty | Sorting [] returns []. |
| Absent | N/A (no keys, no references). |
| Boundary | Sorting [x] returns [x]. Sorting equal elements returns the collection in its original order (stable sort). |
| Partial failure | Sorting produces a fully-sorted result or does not run. |

## Common Mistakes

- Assuming non-empty: writing max = list[0] without checking for empty.
- Ignoring partial failure: a crash mid-write can leave corrupted state.
- Forgetting the identity element: using 0 as the initial value for a product operation.
- Mixing absent and empty: treating a missing key the same as an empty collection.

## Invariants

- Every operation must declare its behavior on empty input and absent references -- silence is a bug.
- The identity element for any accumulation operation must be explicit.
- Partial failure semantics (atomic / idempotent / best-effort) must be documented for every operation that spans multiple steps.
- Boundary values (size 0, size 1, max capacity) must be tested for every operation.

## See Also

- [optionality.md](optionality.md)
- [type-precision.md](type-precision.md)
- [testing/testing.md](../testing/testing.md)
- [patterns/janus-reduction-aggregation.md](../patterns/janus-reduction-aggregation.md)
