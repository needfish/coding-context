# Loop Invariants and Higher-Order Functions

## Loop Invariants

A loop invariant is a condition that holds before every iteration.

```python
# Invariant: max_so_far is the largest value seen in items[0:i]
def find_max(items):
    max_so_far = -inf
    for i in range(len(items)):
        if items[i] > max_so_far:
            max_so_far = items[i]
    return max_so_far
```

## Prefer Higher-Order Functions Over Explicit Loops

Map, filter, fold/reduce, and other combinators eliminate the loop variable, the iteration index, and the mutation.

```python
result = [transform(x) for x in items if condition(x)]
```

| Intent | Combinator | Eliminates |
|---|---|---|
| Transform every element | map / list comprehension | Index variable, accumulator |
| Keep matching elements | filter / list comp with if | Conditional append, index |
| Combine into one value | fold / reduce | Accumulator variable, loop body |
| Check all elements | all | Flag variable, early break |
| Check any element | any | Flag variable, early break |
| Pair adjacent elements | zip(xs, xs[1:]) | Index variable, off-by-one guard |
| Group by key | itertools.groupby | Bucket dict, nested loop |

## When an Explicit Loop Is Needed

Declare the invariant as a comment or assertion before writing the body.

```python
# [invariant]: result is the concatenation of all processed sublists
# [termination]: i advances toward len(nested) each iteration
result = []
for i, sub in enumerate(nested):
    result.extend(sub)
```

Common loop invariants:
- **Prefix processed**: elements [0:i] have been processed, elements [i:] have not.
- **Best so far**: the accumulator holds the best value among elements seen.

Higher-order function first, explicit loop second.

## See Also

- [implementation.md](implementation.md)
- [testing/testing.md](../testing/testing.md)
- [modeling/invariants.md](../modeling/invariants.md)
