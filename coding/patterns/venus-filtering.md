# venus -- filtering

## Problem

A list of items must be reduced to a subset that matches certain criteria. The criteria may be a value range, a property value, a type match, or a compound condition that changes per use case.

## Input

- A list of items
- A predicate function that takes one item and returns true or false

## Output

- A list containing only items where the predicate returned true

## Observable Behavior

- Every item in the output list passes the predicate: checking each item with the predicate returns true
- The filter does not change the items themselves
- The filter does not change the order of items
- Running the filter twice on the same input and predicate produces the same output

## Minimal Code

```python
def filter_items(items, predicate):
    return [item for item in items if predicate(item)]

# Usage:
result = filter_items(all_users, lambda u: u.is_active)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Predicate with side effects | Filtering produces different results on the same input | Make the predicate a pure function |
| Predicate depends on external state (time, random, network) | Same item passes filter one call, fails the next | Capture external state before filtering |
| Predicate always returns false | Filtered result is always empty | Log when filter produces no results |
| Predicate too expensive | Filtering takes too long per item | Pre-compute the filter key or cache results |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Eager vs. lazy filtering | Eager builds a new list in memory. Lazy yields items one at a time using less memory. |
| Single predicate vs. composed predicates | Single is simple but each new criterion needs a new function. Composed (and, or, not) builds complex filters from simple parts. |
| Client-side vs. server-side filtering | Client-side loads all items then filters in memory. Server-side sends the predicate to the database. |

## Start With

The minimal code above. Add composed predicates (and/or/not) when more than one criterion is needed.

## Split Into Its Own Module When

- Filtering logic combines more than one predicate with and/or/not.
- The predicate needs to be configurable by the user (filter UI, query language).
- The same predicate is used across multiple lists or modules.
- Filtering needs optimisation (index-based filtering, predicate pushdown).
- The filtered result is used by multiple consumers.

## Signs This Is Present

- A loop with an if condition selects items from a list.
- A list comprehension or filter() call includes a condition.
- A SQL WHERE clause or API query parameter narrows results.
- A search or find feature filters results based on user input.

## Common Mistakes

- Writing a predicate that changes the list being filtered (concurrent modification).
- Filtering but forgetting to use the result (the original list is unchanged).
- Using a predicate that gives different results for the same input on different calls.
- Filtering multiple times when one pass with a combined predicate would work.
- Not handling the case where all items are filtered out.
- Making the predicate too expensive for the list size.
