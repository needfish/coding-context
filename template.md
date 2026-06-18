# [PLANET NAME]: [Problem Name]

## Problem

One-sentence statement of the recurring problem.

## Context

The setting in which this problem arises. What kind of system, what stage of decomposition, what surrounding structure.

## Domains Involved

The types of domains that participate in this problem, per Jackson's domain taxonomy:

- **Causal** — behaves according to physical or logical laws (e.g., a database, a network socket)
- **Lexical** — a representation of data (e.g., a file, a display buffer, a data structure)
- **Biddable** — behaves unpredictably from the system's perspective (e.g., a human user, an external agent)

List each domain, its type, and its role in the problem.

## Shared Phenomena

The phenomena (events, data, commands) that cross the interfaces between domains. What does each domain contribute? What does each domain observe?

## Invariants

Constraints that must always hold for any valid instantiation of this problem pattern.

## Known Failure Modes

Specific ways this problem type tends to go wrong. These are the characteristic difficulties that any solution must address.

## Forces

The tensions that shape any solution to this problem. Trade-offs inherent in the problem structure — not implementation choices.

## Related Subproblems

Other problem patterns that commonly appear alongside this one. Natural decomposition paths: what subproblems emerge when you analyse this one further.

### ==== Practical Use ====

#### When this pattern matters

#### Start with (minimal form)

The simplest implementation that avoids the pattern's characteristic failure modes.

#### Split into its own module when (observable triggers)

- <symptom that signals the first axis is pushing>
- <symptom that signals a second axis is pushing>
- <symptom that signals a full pattern instantiation is warranted>

#### Common over-engineering traps

- <design mistake people make when they apply this pattern too early>
- <common wrong implementation that looks correct>

## Frame Concern

The central difficulty — Jackson's term. The one thing that makes this problem type non-trivial, regardless of how you implement it.
