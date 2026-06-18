# venus: Filter

> **Pressure class:** Object variation — the predicate shape stays the same, the target collection changes.

## Problem

A collection must be reduced to a subset based on a predicate, discarding elements that do not match.

## Context

Arises whenever a set of items is larger than what the user needs or the next operation can handle. Filtering is rarely a standalone problem — it typically feeds a display (mercury), a transformation (neptune), or a processing pipeline. The predicate may be fixed at design time or supplied dynamically by a user.

## Domains Involved

- **Source collection** (lexical): the full set of items to be filtered. May be in-memory or accessed lazily.
- **Predicate** (lexical or biddable): the criterion for inclusion. May be a static expression, a user-supplied query, or a function computed from other data.
- **Result collection** (lexical): the subset of items that satisfy the predicate.

## Shared Phenomena

- Source → Filter: items, item count
- Predicate → Filter: matching function, parameters
- Filter → Result: subset, count of discarded items
- Result → Consumer: items (when enumerated)

## Invariants

- Every element in the result must satisfy the predicate.
- Every element in the source that satisfies the predicate must appear in the result.
- The filter must not modify or duplicate elements — it selects, it does not transform.

## Known Failure Modes

- **Side effects**: The predicate mutates state or produces external effects during evaluation.
- **Non-determinism**: The predicate returns different results for the same element on different evaluations, leading to inconsistent subsets.
- **Performance**: Filtering a large collection with an expensive predicate blocks the caller beyond acceptable latency.
- **Order dependence**: The predicate behaviour depends on previously filtered elements (e.g., stateful predicates).
- **All-or-nothing errors**: A predicate that throws on one element aborts the entire filter, discarding valid results.

## Forces

- **Eager vs. lazy evaluation**: Eager filtering materialises the result immediately (predictable cost); lazy filtering defers work until elements are consumed (lower peak cost, harder to predict).
- **Generality vs. optimisation**: A generic predicate handles any criterion but cannot exploit index or sort order of the source.
- **Strictness vs. fault tolerance**: A strict filter aborts on the first predicate failure; a fault-tolerant filter skips failing elements.

## Related Subproblems

- **mercury/display**: Filtered results are commonly displayed.
- **earth/sort**: Filtering and sorting often compose — filter first, then sort (or vice versa, depending on selectivity).
- **neptune/transform**: Each element in the result may need transformation.
- **jupiter/validate**: The predicate itself may need validation before use.
- **mars/persist**: The source collection may come from storage.

### ==== Practical Use ====

#### When this pattern matters
Any project that selects a subset from a larger collection — search, browse, list, query, report.

#### Start with (minimal form)
A `for` loop with an `if` inside. Python: `[x for x in items if condition]`. Rust: `items.iter().filter(|x| predicate(x))`. A single function that takes the collection and returns the subset.

#### Split into its own module when (observable triggers)
- The predicate becomes user-configurable (dynamic, not hard-coded).
- Multiple filter criteria compose (AND/OR/NOT combinations).
- The source collection is large enough that lazy/streaming filtering matters.
- The filtering logic needs independent testing from the rest of the pipeline.

#### Signs you're in this pattern
- Your code has a loop with an `if` inside that decides what to keep or discard.
- The predicate condition appears in multiple places, slightly different each time.
- You're passing a collection through a function that returns a smaller collection of the same type.
- Filter logic is interleaved with display logic, making each harder to change independently.

#### Real project examples
- **crawley** (`extractor` module): URL link extraction filters parsed HTML by element type (img, script, video) — a predicate per resource type.
- **smoltcp** (socket dispatch): TCP socket matching filters incoming packets by port/protocol — a predicate per socket type.
- **MiniJinja** (`filters.rs`): Template filter chain applies predicate tests (defined, string length, regex match) to values before rendering.

#### Common over-engineering traps
- Building a predicate parser/interpreter before measuring that the fixed predicates are the bottleneck.
- Abstracting the filter into a generic pipeline stage before you have a second pipeline stage.
- Implementing parallel filtering before profiling shows single-threaded is too slow.

## Frame Concern

**Predicate ownership.** The predicate and the filter execute in different contexts — the predicate may hold state, access external resources, or change over time. The filter must handle a predicate that is not trustworthy for the domain it inspects.
