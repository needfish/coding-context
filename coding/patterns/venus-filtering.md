# ------------------------------------------------------------------------------
#  venus-filtering.md
#  Object variation -- the predicate shape stays the same, the target collection changes.
# ------------------------------------------------------------------------------

# - Problem -

You have a list of items and you want only the ones that match a certain condition. Keep the matches, discard the rest.

# - When You'll See This -

Any time you select a subset from a larger collection -- search results, browsing a directory, filtering a list, building a report query. Filtering usually feeds into display, transformation, or a pipeline.

# - How It Works -

- **Source collection**: the full list of items you're filtering from.
- **Predicate**: the rule that decides what stays and what goes. Could be a hard-coded check or something a user types in.
- **Result collection**: the smaller list of items that passed the check.

# - What Must Always Be True -

- Every item in the result satisfies the predicate.
- Every item from the source that satisfies the predicate shows up in the result.
- Filtering only selects -- it doesn't change or duplicate items.

# - What Goes Wrong -

- **Predicate with side effects**: the filter condition modifies something or makes network calls while checking.
- **Non-deterministic predicate**: the same item passes the check sometimes and fails other times.
- **Slow predicate on big data**: checking each item takes too long because the condition is expensive.
- **Stateful predicate**: the filter result depends on which items were checked before (like a running average).
- **One bad apple crashes everything**: a predicate that throws on one item kills the whole filter pass.

# - Design Trade-Offs -

- **Eager vs. lazy**: process everything now (predictable cost) or wait until someone reads the results (lower peak, harder to predict).
- **Generic vs. optimised**: a general predicate handles any condition but can't exploit sorted or indexed data.

# - Related Problems -

- **mercury/display**: filtered results usually get shown to someone.
- **earth/sorting**: filter then sort (or sort then filter, depending on which is cheaper).
- **neptune/transformation**: each filtered item might need processing.
- **janus/reduction**: count or sum the matching items.
- **jupiter/validation**: the filter rule itself might need checking.
- **mars/persistence**: the source data might come from disk.

# - Getting Started -

A `for` loop with an `if` inside. Python: `[x for x in items if condition]`. Rust: `items.iter().filter(|x| predicate(x))`. One function that takes a collection and returns the subset.

# - When to Split Into Its Own Module -

- Users can configure the filter condition at runtime.
- You need AND/OR/NOT combinations of multiple filters.
- The source is large enough that lazy streaming matters.
- Filter logic needs its own tests separate from the rest of the pipeline.

# - Signs You're in This Pattern -

- A loop with an `if` inside decides what to keep or discard.
- The same filter condition appears in multiple places with tiny differences.
- You pass a collection to a function and get back a smaller collection of the same type.
- Filter logic is mixed with display logic, making each harder to change.

# - Common Mistakes -

- Building a filter expression parser before measuring that hard-coded conditions are the bottleneck.
- Abstracting filter into a generic pipeline stage before you have a second stage.
- Adding parallel filtering before profiling shows single-threaded is too slow.

# - The Core Challenge -

The predicate and the filter run in different contexts. The predicate might hold state, access external resources, or behave differently over time. The filter has to work with a predicate it can't fully trust.
