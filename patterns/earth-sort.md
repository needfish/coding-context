# earth: Sort

## Problem

A collection must be ordered by a key derived from each element, establishing a total order among items.

## Context

Arises whenever items have a natural or user-defined ordering and the consumer requires ordered access. Sorting is rarely the terminal subproblem — sorted collections feed display (mercury), further processing (neptune), or storage (mars/persist). The sort key may be a single field, a compound key, or a computed function.

## Domains Involved

- **Source collection** (lexical): unordered or partially ordered items.
- **Comparator** (lexical): a function or key extractor that defines the ordering relation.
- **Ordered collection** (lexical): items in the established total order.

## Shared Phenomena

- Source → Sort: items
- Comparator → Sort: comparison function or key extractor
- Sort → Ordered: items in sequence

## Invariants

- For any two distinct elements a, b in the source, either a precedes b or b precedes a in the result (totality).
- The ordering is transitive: if a precedes b and b precedes c, then a precedes c.
- The ordered collection contains exactly the same elements as the source collection.
- Sorting is stable if elements with equal keys preserve their original relative order.

## Known Failure Modes

- **Non-transitive comparator**: The comparison violates transitivity (a < b, b < c, c < a), causing garbage output or non-termination.
- **Comparator side effects**: The comparator mutates state or produces external effects during comparison.
- **Performance cliff**: Sorting an already-sorted collection with a naive comparator is O(n²) — the worst implicit assumption.
- **Partial order mishandling**: A comparator that returns "equal" for distinct elements when the domain has only a partial order.
- **Memory pressure**: Sorting in memory when the collection exceeds available memory.

## Forces

- **Stability**: Maintaining original order for equal keys adds memory overhead but is essential for multi-key sorts.
- **In-place vs. copying**: In-place sorting reduces memory but constrains concurrency and aliasing.
- **Comparison cost**: An expensive comparator favours algorithms that minimise comparisons (merge sort, tournament sort); a cheap comparator favours algorithms with good cache behaviour (quicksort, timsort).
- **Adaptivity**: The collection may be nearly sorted (common in practice) — an adaptive sort runs in O(n) on nearly-sorted input.

## Related Subproblems

- **venus/filter**: Sort often follows or precedes filter.
- **mercury/display**: Ordered results are typically displayed.
- **neptune/transform**: Sort keys may require transformation before comparison.
- **mars/persist**: Sorted results may be persisted.

## Frame Concern

**Comparator consistency.** The comparator must define a valid total order on the domain, must be consistent across multiple invocations, and must not mutate. The sort algorithm depends on this contract being met — but the comparator is supplied externally and may violate it.
