# janus: Reduction / Aggregation

> **Pressure class:** Behavior variation — the combining operation changes while the accumulate-over-elements contract stays the same.

## Problem

A collection of elements must be combined into a single accumulated value by repeatedly applying a binary operation, starting from an initial value (or using the first element as the seed).

## Context

Arises whenever a system needs a summary, total, extremum, concatenation, or composite derived from a collection. The operation is applied pairwise across all elements — each element produces an intermediate result that feeds into the next application. The final accumulated value is the single output. This is distinct from transformation (neptune), which maps each element independently, and from filtering (venus), which selects a subset. Reduction is a terminal subproblem — it typically consumes the output of filtering (venus) or transformation (neptune) and produces the final result.

## Domains Involved

- **Source collection** (lexical): the set of elements to be reduced. May be finite or infinite (streaming reduce requires early termination).
- **Binary operation** (causal): the function that combines two elements into one. Must be deterministic for repeatable results.
- **Accumulator** (lexical): the intermediate result carried between element visits. Starts as the initial value (or first element) and is updated by each application of the binary operation.
- **Initial value** (lexical): the starting value for the accumulator. Must be the identity element of the binary operation for correctness with empty collections.
- **Result** (lexical): the final single value produced after processing all elements.

## Shared Phenomena

- Source collection → Reduce: elements in sequence (or unordered)
- Binary operation → Reduce: function that takes (accumulator, element) → new accumulator
- Initial value → Reduce: starting accumulator value
- Reduce → Result: final accumulator after all elements processed

## Invariants

- The binary operation must be deterministic — the same inputs must always produce the same output.
- The initial value must be the identity element for the binary operation: `op(identity, x) = op(x, identity) = x` for all valid `x`.
- An empty collection with an initial value must return the initial value.
- Reducing a collection with one element must be equivalent to applying the operation to the initial value and that single element.
- The result must be independent of traversal order if the operation is associative.
- The binary operation must not mutate the input elements or the accumulator in ways visible outside the reduction.

## Known Failure Modes

- **Missing identity element**: An empty collection is reduced without an initial value, causing a runtime error (e.g., `max([])` throws).
- **Non-associative operation**: String concatenation is associative; floating-point addition is not. Different fold orders produce different results for the same data.
- **Side effects in the operation**: The binary function mutates external state or the input elements, producing unexpected behavior on retry or parallel execution.
- **Accumulator type mismatch**: The operation returns a different type from the source elements (e.g., counting — collection of strings, accumulator is integer), but the initial value is missing or wrong-type.
- **Left vs. right fold asymmetry**: The operation is not associative, so `foldl` and `foldr` produce different results, and the caller uses the wrong one.
- **Early termination**: An infinite or very large collection is reduced without a mechanism to stop early (e.g., `any` can short-circuit but a generic reduce cannot).
- **Overflow**: Numeric accumulation exceeds the accumulator type's range without detection.
- **Empty collection blame**: The reduction fails on an empty collection, but the root cause is upstream (an empty result from filtering or an empty input file).

## Forces

- **Left fold vs. right fold**: Left fold processes elements from the first to the last, accumulating iteratively — natural for iterative evaluation, stack-safe. Right fold processes from the last backward, nesting applications — natural for lazy evaluation, requires stack space proportional to collection size for eager evaluation.
- **Associative vs. ordered**: Associative operations (sum, max, set union) can be parallelised and reordered; non-associative operations (string concatenation, list append) must preserve order and are sequential.
- **Eager vs. lazy accumulation**: Eager reduction materialises the result immediately (predictable cost, blocks until done). Lazy/streaming reduction emits an accumulated value progressively (lower peak memory, harder to reason about final state).
- **Generic vs. specialised**: A generic `reduce` function handles any binary operation but is harder to optimise. Specialised forms (`sum`, `max`, `count`) are simpler and faster but cover fewer cases.
- **Early termination vs. full traversal**: Some reductions can stop early (``any``: stop at first `true`; ``max``: must traverse all). A generic reduce forces full traversal even when the operation could short-circuit.

## Related Subproblems

- **venus/filtering**: Elements are often filtered before reduction.
- **neptune/transformation**: Elements are often transformed before reduction (map-reduce pattern).
- **earth/sorting**: For non-associative operations, sort order determines the reduce result.
- **mars/persistence**: Reduction results are often persisted as summaries or caches.
- **mercury/display**: Reduction results are terminal — they feed directly into display.
- **eris/pipeline**: Reduce is often the final stage of a pipeline.

### ==== Practical Use ====

#### When this pattern matters
Any project that computes a summary over a collection — totals, averages, maxima, minima, concatenations, counts, boolean conjunctions/disjunctions, building lookup maps, computing histograms.

#### Start with (minimal form)
A `for` loop with an accumulator variable. Python: `total = 0; for x in items: total += x`. Rust: `let mut sum = 0; for x in &items { sum += x; }`. No abstraction around the reduce itself — the loop is the reduce.

```python
total = 0
for x in items:
    total += x
```

```rust
let mut sum = 0i64;
for x in &items {
    sum += *x;
}
```

#### Split into its own module when (observable triggers)
- The same accumulation pattern appears in multiple places (sum columns, count matching, max per group).
- The binary operation needs to be parameterised (different aggregations for different callers).
- The operation is non-associative and fold direction matters (left vs. right).
- The collection is large enough that streaming or parallel reduction matters.
- An empty collection causes errors because no identity element is defined.

#### Signs you're in this pattern
- You have a `for` loop with an accumulator variable that persists across iterations.
- The same accumulation logic appears in multiple places with slight variations.
- A function takes a collection and returns a single value computed from all elements.
- The computation can be expressed as `reduce(operation, initial, collection)`.

#### Real project examples
- **tokei** (`src/stats.rs`): Language stats are reduced from per-file counts into per-language totals — sum of lines, max of file counts, concatenation of file paths.
- **micrograd** (`engine.py`): Backpropagation reduces gradient contributions from multiple paths into a single accumulated gradient per node — sum of partial derivatives.
- **crawley** (URL extraction): Extracted links from all pages are reduced into a single deduplicated set — set union across pages.
- **tiny-lsm-go** (`compaction.go`): Multiple SST files are reduced into one merged file during compaction — merge of sorted runs.

#### Common over-engineering traps
- Writing a generic `reduce()` function before you have a second reduction pattern (just write the `for` loop).
- Using a fold when the operation has no identity element and empty collections must be handled specially.
- Assuming associativity for operations that are not associative (floating-point arithmetic, subtraction, division).
- Building a parallel reduce framework before measuring that single-threaded reduction is the bottleneck.
- Extracting reduce to a higher-order function before you have a second operation to pass.

## Frame Concern

**Associativity and the identity element.** The binary operation defines how two values combine, but the reduction's correctness depends on properties of that operation that the reduce implementation cannot enforce. If the operation is not associative, different evaluation orders produce different results. If the identity element is wrong or absent, empty collections cause errors or produce nonsensical values. The concern is not how to apply the operation repeatedly — that is mechanical — but how to ensure the operation's algebraic properties match the caller's expectations, and what happens when they do not.
