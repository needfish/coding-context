# janus -- reduction / aggregation

## Problem

A list of values must be combined into a single value using a combining operation: sum, product, max, min, AND, OR, string concatenation, count, average, histogram. The combining operation must work on two values at a time and the order of combining must not change the result (associative).

## Input

- A list of values
- A combining operation that takes two values and returns one value
- An identity element for the operation (the result when the list is empty)

## Output

- A single combined value (or the identity element if the list is empty)

## Observable Behavior

- The combining operation is applied pairwise until one value remains
- The order of combining does not change the result (associative property)
- An empty list returns the identity element (does not crash)
- The combining operation does not change the input values (no side effects)
- The same input and operation always produce the same result

## Minimal Code

```python
from functools import reduce

total = sum(numbers)          # addition, identity: 0
product = reduce(lambda a, b: a * b, numbers, 1)  # multiplication, identity: 1
max_val = max(numbers) if numbers else float("-inf")
all_active = all(items)       # boolean AND, identity: true
any_active = any(items)       # boolean OR, identity: false
```

## Identity Element Reference

| Operation | Identity element |
|---|---|
| Addition | 0 |
| Multiplication | 1 |
| Max | -infinity |
| Min | +infinity |
| String concatenation | "" |
| List concatenation | [] |
| Boolean AND | true |
| Boolean OR | false |
| Set union | {} (empty set) |
| Count | 0 |

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Wrong identity element | Using 0 as identity for multiplication: result is always 0 | Use 1 as identity for multiplication |
| Non-associative operation | Subtraction as reduction: (10 - 5) - 2 != 10 - (5 - 2) | Document that operation must be associative, or use explicit left/right fold |
| Overflow | Summing large integers overflows | Use a wider type (long, big integer) or saturating arithmetic |
| Floating-point error | Summing many floats accumulates rounding error | Use Kahan summation or sort by magnitude first |
| Empty collection crash | Reduction crashes because the list is empty | Return the identity element for empty collections |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Sequential vs. parallel | Sequential is simple but uses one core. Parallel (divide the list, reduce each part, combine results) needs the operation to be associative. |
| Eager vs. lazy (streaming) | Eager processes the entire list at once. Lazy processes elements as they arrive, memory-efficient but needs incremental accumulation. |
| Exact vs. approximate | Exact (sum, count) is precise but may be slow for large datasets. Approximate (HyperLogLog for distinct count, reservoir sampling for median) is fast but has error bounds. |

## Start With

Use the language's built-in sum, max, min, all, any, or reduce. Only write a custom reduction when the operation is not one of the standard ones.

## Split Into Its Own Module When

- Custom reduction operations beyond simple sum/max are needed (median, mode, histogram).
- The reduction must be parallelised for performance.
- The reduction is streaming (elements arrive over time).
- Multiple reductions are computed from the same list.

## Signs This Is Present

- A loop accumulates a value: total += x, count += 1, result = result + x.
- reduce(), fold(), or aggregate() is used.
- SUM, COUNT, AVG, GROUP BY queries are constructed.
- A running total or cumulative statistic is computed.

## Common Mistakes

- Using a non-associative operation (subtraction, division, string concatenation with separators).
- Forgetting the identity element and crashing on empty input.
- Using floating-point accumulation for financial calculations (use decimal or integer cents).
- Parallelising a reduction without ensuring the operation is associative and thread-safe.
- Computing the same reduction multiple times over the same list.
