# ------------------------------------------------------------------------------
#  janus-reduction-aggregation.md
#  Behavior variation -- the combining operation changes while the accumulate-over-elements contract stays the same.
# ------------------------------------------------------------------------------

# - Problem -

You have a collection of items and you need to combine them into a single value -- a sum, a product, a maximum, a concatenation, or any operation that repeatedly applies a binary function across elements.

# - When You'll See This -

Any time you need a summary, total, count, extremum, or composite derived from a collection. Summing prices, finding the longest name, concatenating strings, counting matches. Reduction is usually the terminal step -- it consumes the output of filtering or transformation and produces the final result.

# - How It Works -

- **Source collection**: the items to combine, each with a value or attribute.
- **Binary operation**: the function that combines two values into one (addition, max, string concatenation, boolean AND).
- **Initial value**: the starting point for the accumulation (0 for sum, 1 for product, empty string for concatenation, -inf for max).
- **Accumulated result**: the single output value after processing all elements.

# - What Must Always Be True -

- The binary operation is associative: `op(a, op(b, c)) == op(op(a, b), c)` for all a, b, c.
- The initial value is the identity element for the operation (0 for addition, 1 for multiplication).
- The operation has no side effects.
- Every element in the source contributes to the result exactly once.
- Reducing an empty collection returns the initial value.

# - What Goes Wrong -

- **Non-associative operation**: floating-point addition is not associative, so parallel reduction gives different results than sequential.
- **Wrong initial value**: starting max at 0 means negative values are never selected.
- **Side effects in operation**: the combining function mutates shared state.
- **Overflow**: summing a large collection exceeds the maximum representable value.
- **Empty collection returns nothing**: no initial value means empty input produces no result or an error.

# - Design Trade-Offs -

- **Sequential vs. parallel reduction**: sequential is simple and deterministic; parallel is faster for large collections but requires associativity and a merge step.
- **Eager vs. lazy**: eager reduction computes immediately (predictable); lazy defers until the result is needed.
- **Order-dependent vs. order-independent**: order-dependent reduction (string concatenation) is simpler; order-independent (sum, max) enables parallelism.

# - Related Problems -

- **venus/filtering**: reduced values often come from filtered subsets.
- **neptune/transformation**: elements may need transformation before reduction.
- **earth/sorting**: sorted elements might feed into a ranked reduction.
- **mercury/display**: reduced results (totals, averages, counts) are commonly displayed.

# - Getting Started -

A loop with an accumulator variable. Python: `total = 0; for x in items: total += x`. Rust: `items.iter().sum()`. Use your language's built-in `sum`, `max`, `reduce`, or `fold`.

# - When to Split Into Its Own Module -

- The reduction operation is complex (multi-field aggregation, grouping, filtering within the reduce).
- You need multiple kinds of reduction on the same data (sum, count, average).
- You need parallel reduction for performance.
- The reduction logic needs independent testing.

# - Signs You're in This Pattern -

- You have a loop that accumulates a value by processing each element.
- You're computing a total, average, count, maximum, or concatenation.
- The accumulation logic is mixed with element transformation.
- You need to compute multiple aggregates from the same collection.

# - Common Mistakes -

- Writing a custom reduce when Python's `sum()` or `functools.reduce()` works.
- Forgetting the initial value for empty collections.
- Using floating-point addition where order independence (and thus parallelism) matters.
- Putting side effects (print, logging, network calls) inside the combining function.

# - The Core Challenge -

Reduction depends on associativity for correctness and parallelism. The operation must combine elements in any order and produce the same result. Floating-point arithmetic, string concatenation, and many real operations are not truly associative, which limits how you can parallelise and requires careful ordering guarantees.
