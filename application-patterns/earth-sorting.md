# earth -- sorting

## Problem

A list must be ordered by a comparison rule that changes per use case: by name, by date, by priority, by a custom metric. The sort must produce a predictable order and equal items must stay in their original relative order (stable sort).

## Input

- A list of items
- A comparison function or sort key: takes two items and returns which comes first, or extracts a key from each item

## Output

- The same items in sorted order (either a new list or in-place rearrangement)

## Observable Behavior

- Sorting the same list with the same comparison always produces the same result
- Equal items keep their original relative order (stable sort)
- The comparison function is called many times during sorting
- The sort completes in O(n log n) time

## Minimal Code

```python
users.sort(key=lambda u: u.name)
# or for multiple keys:
users.sort(key=lambda u: (u.last_name, u.first_name))
```

```typescript
users.sort((a, b) => a.name.localeCompare(b.name));
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Inconsistent comparator | Comparator returns different results for same pair across calls; sort produces wrong order, crashes, or never terminates | Ensure comparator is a pure function that does not depend on external state |
| Non-transitive comparator | a < b and b < c but a > c; sort produces wrong order | Verify transitivity with property-based testing |
| Unstable sort breaks expectations | Equal items appear in different order each run | Use a stable sort (MergeSort, TimSort) or add a tie-breaking secondary key |
| Comparator with side effects | Comparator modifies the items being compared | Use a pure key extraction function; compute all keys before sorting |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Stable vs. unstable | Stable (MergeSort, TimSort) preserves order of equal items but uses more memory. Unstable (QuickSort, HeapSort) is in-place but may reorder equal items. |
| Comparison-based vs. radix | Comparison sorts work on any ordered type but are O(n log n). Radix sorts are O(n * k) but need keys decomposable into digits. |
| In-place vs. copying | In-place uses O(1) extra memory but changes the original list. Copying uses O(n) memory but leaves the original unchanged. |

## Start With

Use the language's built-in sort method with a key function. Add a custom comparator function only when the built-in key mechanism cannot express the ordering.

## Split Into Its Own Module When

- The comparator logic becomes complex (tie-breaking, multiple sort keys, direction per key).
- Sorting needs to be customised at runtime by the user (clickable column headers).
- The collection is too large for an in-memory sort and needs external sorting.
- A custom sort algorithm is needed for performance.

## Signs This Is Present

- A list is sorted with a lambda or custom comparator.
- Columns in a table are clickable to sort ascending/descending.
- Multiple sort criteria are combined (sort by date, then by priority).
- A sort key is computed from multiple fields with tie-breaking rules.

## Common Mistakes

- Using a comparator that changes the objects being compared (side effects).
- Assuming the sort is stable when it is not.
- Using locale-sensitive comparison in a hot loop (expensive -- pre-extract the sort key).
- Sorting on a field that can change during the sort.
- Forgetting that sorting descending (reverse) is a separate operation from sorting ascending.
