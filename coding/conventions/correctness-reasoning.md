# ------------------------------------------------------------------------------
#  correctness-reasoning.md
#  Loop invariants, higher-order functions, and reasoning about code
#  correctness.
# ------------------------------------------------------------------------------

# Loop Invariants and Higher-Order Functions

## Loop Invariants

A **loop invariant** is a condition that holds before every iteration. It is
the tool for reasoning about correctness of an imperative loop:

```python
# Invariant: max_so_far is the largest value seen in items[0:i]
def find_max(items):
    max_so_far = -inf          # invariant holds trivially (empty prefix)
    for i in range(len(items)):
        if items[i] > max_so_far:
            max_so_far = items[i]   # invariant restored
        # invariant holds at end of each iteration
    return max_so_far               # invariant implies result is correct
```

## Prefer Higher-Order Functions Over Explicit Loops

**Prefer higher-order functions over explicit loops with invariants.**
Map, filter, fold/reduce, and other combinators eliminate the loop variable,
the iteration index, and the mutation -- which means no loop invariant to
write or debug:

```python
# Loop with invariant:
result = []
for x in items:          # invariant: result contains all processed items so far
    if condition(x):
        result.append(transform(x))

# Same logic without loop variables:
result = [transform(x) for x in items if condition(x)]
```

When a combinator directly expresses the intent, use it:

| Intent | Combinator | Eliminates |
|---|---|---|
| Transform every element | `map` / list comprehension | Index variable, accumulator |
| Keep matching elements | `filter` / list comp with `if` | Conditional append, index |
| Combine into one value | `fold` / `reduce` | Accumulator variable, loop body |
| Check all elements | `all` | Flag variable, early break |
| Check any element | `any` | Flag variable, early break |
| Pair adjacent elements | `zip(xs, xs[1:])` | Index variable, off-by-one guard |
| Group by key | `itertools.groupby` | Bucket dict, nested loop |

## When You Need an Explicit Loop

**When you need an explicit loop,** declare the invariant as a comment or
assertion before writing the body. The invariant tells you what must be true
at the start of each iteration, which makes the loop body's logic fall out
naturally:

```python
# [invariant]: result is the concatenation of all processed sublists
# [termination]: i advances toward len(nested) each iteration
result = []
for i, sub in enumerate(nested):
    result.extend(sub)
```

The two most common loop invariants are:

- **Prefix processed**: elements `[0:i]` have been processed, elements `[i:]`
  have not. Used in linear scans, searches, and filters.
- **Best so far**: the accumulator holds the best value among elements seen.
  Used in max, min, and extreme-finding.

**Higher-order function first, explicit loop second.** Start with a
combinator. Only reach for an explicit loop with an invariant when the
computation doesn't fit a standard combinator -- for example, updating a
running state that depends on the previous iteration's output, or
traversing a structure where the order of processing is data-dependent.

## See Also

- [implementation.md](implementation.md) -- main implementation process
- [testing/testing.md](../testing/testing.md) -- evaluating invariants through testing
- [modeling/invariants.md](../modeling/invariants.md) -- categories of invariants and encoding strategy
