# Testing

## Test Structure

- Mirror source in tests/. src/user/auth.py becomes tests/test_user_auth.py.
- Name tests test_<unit>_<scenario>.
- Test public API contracts, edge cases, and error paths. Focus on behavior.
- Structure tests as Arrange, Act, Assert. Use one assertion per test where practical.

## What to Test

### Edge Cases

Test with:
- Empty collection
- Single-element input
- Structurally invalid input (wrong type, missing field, out-of-range value)
- Maximum-size input at the domain boundary
- Optional/absent values -- for each optional field, test the present and absent cases. See [modeling/optionality.md](../modeling/optionality.md).

Add guard clauses that return Err or raise at function entry for each invalid state.

### Data Representation Walkthrough

For data representations, walk through each operation on the decomposition.

| Property | Check |
|---|---|
| Empty | Root exists with no children. |
| One | Insertion creates one instance of each node per path. Removal cleans up all and only those. |
| Many | Containers scale. A hash table maintains O(1); a linked list degrades scan. |
| Domain | What happens when a field is zero, negative, absent, or outside range. |
| Concurrency | Coarse locks serialize but are simple. Fine-grained locks need a total order to prevent deadlock. |
| Cost | Point lookup O(1) hash / O(log n) tree. Scan O(k + log n). Insert O(1). Remove O(k) to find the cut, O(1) per edge. |

### Property-Based Testing

When the domain has clear invariants: define the invariant, let the framework generate counterexamples. Prefer this over hand-writing every edge case. Fall back to example-based testing for purely sequential or IO-bound logic.

## Known Failure Modes

Open each pattern's "Known Failure Modes" from [patterns/](../patterns/) and test against them.

## Log Surprises

Log surprises. Document what breaks.
