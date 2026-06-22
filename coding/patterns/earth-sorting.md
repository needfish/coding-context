# ------------------------------------------------------------------------------
#  earth-sorting.md
#  Behavior variation -- the comparator algorithm changes per sort key.
# ------------------------------------------------------------------------------

# - Problem -

You have a collection of items that need to be in a specific order. You need to arrange them by some key -- alphabetically, by number, by date, or by a custom rule.

# - When You'll See This -

Any time items need to be presented, processed, or stored in a defined order -- ranked lists, sorted tables, ordered output, priority queues. Sorting feeds into display, further processing, or storage.

# - How It Works -

- **Source collection**: the items in whatever order they happen to be in.
- **Comparator**: the rule that decides which item comes first (by name, by date, by score).
- **Ordered collection**: the same items, now in the right sequence.

# - What Must Always Be True -

- For any two items, one comes before the other in the result (total order).
- If A comes before B and B comes before C, then A comes before C (transitive).
- The sorted collection has exactly the same items as the original.
- Equal elements keep their original relative order if you need stable sorting.

# - What Goes Wrong -

- **Broken comparator**: A < B, B < C, but C < A -- the sort produces garbage or never finishes.
- **Comparator with side effects**: the comparison function mutates state or makes network calls.
- **Hidden O(n^2)**: sorting an already-sorted list with a bad comparator is way slower than it should be.
- **Partial order confusion**: the comparator says two different items are "equal" when they're not.

# - Design Trade-Offs -

- **Stable vs. unstable**: keeping original order for equal items costs extra memory but matters for multi-key sorts.
- **In-place vs. copying**: sorting in place saves memory but makes concurrency harder.
- **Cheap vs. expensive comparisons**: an expensive comparator favours algorithms that minimise comparisons (merge sort); a cheap one favours cache-friendly algorithms (quicksort, timsort).
- **Adaptive sorting**: real-world data is often nearly sorted -- an adaptive sort runs in O(n) on almost-ordered input.

# - Related Problems -

- **venus/filtering**: sort often combines with filter.
- **mercury/display**: ordered results usually get displayed.
- **neptune/transformation**: sort keys might need processing before comparison.
- **mars/persistence**: sorted results might get saved.
- **janus/reduction**: sorted elements often feed into a ranked result.

# - Getting Started -

Use your language's built-in sort: Python `sorted(items, key=...)`, Rust `items.sort_by_key(...)`, JavaScript `items.sort(...)`. Pass a simple key-extraction function.

# - When to Split Into Its Own Module -

- You need multiple sort keys (primary, secondary, tertiary).
- The sort key is expensive to compute -- cache it.
- The sort direction or key is configurable at runtime.
- The collection is too large to fit in memory (external sort).

# - Signs You're in This Pattern -

- Your collection needs ordering and the default order isn't what you want.
- The comparison logic involves multiple fields, some ascending and some descending.
- Different parts of your code need different orderings of the same data.
- You see the same key-extraction function copied in several places.

# - Common Mistakes -

- Writing a custom sort algorithm before proving the built-in sort is too slow.
- Implementing stable sort when element identity doesn't matter.
- Pre-sorting data that the consumer will re-sort anyway.

# - The Core Challenge -

The comparator defines a contract the sort algorithm depends on (total order, consistency, no side effects). But the comparator comes from outside -- it might violate that contract. You need a valid, consistent, pure comparison function.
