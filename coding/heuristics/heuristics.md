# Heuristics

## Starting & Decomposition

- Model nouns as data structures and verbs as functions.
- Hardcode a single use path, then fill in real logic after review. Replace stubs from the leaves up.

## Data Structures

- Store each fact once. Mark derived copies as non-authoritative.
- Give entities IDs. Treat value objects as interchangeable by attributes.
- Use dict/set when lookups outnumber inserts by more than 10 to 1.

## Code Organisation

- Keep everything in one file until a concrete need forces a split.
- Group by feature. Refactor after observing actual volatility (Common Closure Principle).
- Split around domain concepts, not technical layers.
- Extract only when circular dependencies, an invariant needing encapsulation, or the same pattern repeats at two sites.
- Use comments for why, not what.

## Navigating Unknowns

- Build the cheapest version of the riskiest assumption. Measure the result. Keep or discard it.
- Hide uncertain parts behind interfaces; replace later.

## Abstraction

- List facts, then merge by shared key. Record types emerge.
- Group code that changes together.
- Use access-pattern weighting and noun-verb extraction.
- List everything that can go wrong. Each becomes a named interface slot.
- Write the ideal call, then implement backwards.
- Step through the interaction. Roles emerge from participant needs.
- Build a minimal working version. Abstractions emerge from iteration.
- Assign responsibilities. Roles become types.
