# mercury: Display

> **Pressure class:** Data variation — same rendering pipeline, different values to display.

## Problem

Data from a source must be presented to a user in a readable form, potentially involving transformation, formatting, and timely updates.

## Context

Occurs whenever a system has internal data that a human needs to see. The data may be structured (records, tables, trees) or unstructured (text, images, streams). The user may be an operator, a customer, or another system interpreted as biddable. Display is rarely alone — it couples to a data source (mars/persist or neptune/transform) and often precedes a command path (jupiter/validate, saturn/route).

## Domains Involved

- **Source** (lexical or causal): provides the raw data to be displayed. May be a database (causal — queried on demand), a stream (causal — pushes updates), or an in-memory structure (lexical — polled).
- **Display surface** (lexical): the rendering target. A terminal, a GUI widget, an HTML element, a file. Receives formatted representations.
- **User** (biddable): interprets the display. May react by issuing commands, which feeds other problem patterns.

## Shared Phenomena

- Source → Display: data values, timestamps, schema descriptors
- Display → Surface: formatted output (text, pixels, DOM nodes)
- User → System: commands or queries triggered by interpretation of the display
- Surface → Display: events (resize, scroll, focus) that affect what can be shown

## Invariants

- Every displayed value must correspond to a source value at some identifiable snapshot time.
- Format transformations must be lossless with respect to the information the user needs (loss may be acceptable for information the user does not need).
- The display must update within the user's perceptual threshold for the domain (sub-second for interactive, seconds for monitoring, minutes for dashboards).
- The display must not present contradictory information from the same source at the same time.

## Known Failure Modes

- **Aliasing / staleness**: Display shows a value that no longer reflects the source state. The user acts on outdated information.
- **Overflow**: The data quantity exceeds the display surface capacity. Information is truncated or omitted without indication.
- **Inconsistent framing**: Different parts of the display update from different snapshots, showing contradictory states.
- **Format loss**: Transformation discards information the user needs (e.g., truncating decimals in a financial display).
- **Latency mismatch**: The display update cycle is slower than the source change cycle, causing visible flicker or missed states.
- **Semantic mismatch**: The representation (colour, position, icon) implies a meaning not present in the data.

## Forces

- **Fidelity vs. timeliness**: Rendering every detail accurately takes time; the user may prefer a fast approximate display.
- **Breadth vs. depth**: Showing all information at once overwhelms the user; hiding information requires navigation that may obscure context.
- **Stability vs. responsiveness**: Smoothing updates avoids flicker but introduces latency.
- **Generality vs. specificity**: A generic display framework handles many cases but may not handle any case well.

## Related Subproblems

- **mars/persist**: The source data may come from a persistence layer.
- **neptune/transform**: Raw data often requires transformation before display.
- **jupiter/validate**: User commands triggered by display interpretation must be validated.
- **saturn/route**: Display events may need routing to handlers.
- **earth/sort**: Ordered display requires a sorting subproblem.
- **venus/filter**: Subset display requires a filtering subproblem.

### ==== Practical Use ====

#### When this pattern matters
Any project that shows data to a human — CLI tools with output, dashboards, editors, games, monitoring UIs.

#### Start with (minimal form)
Print raw values to stdout with `println!` / `print()` / `console.log()`. Format as a table with simple string padding. Refresh by re-rendering the entire output on each update.

#### Split into its own module when (observable triggers)
- Multiple output targets (terminal, file, JSON, HTML) need different renderers.
- The source data updates faster than the display refresh cycle — you need throttling or diffing.
- The user can interact with the display (scroll, click, filter) — events must route back to data.
- The display needs to survive process restart (session state, window layout).

#### Common over-engineering traps
- Building a widget/compositor framework before you have more than one view type.
- Separating display logic from data fetching before you have a second data source.
- Implementing virtual scrolling before you measure that rendering all items is a bottleneck.

## Frame Concern

**The temporal coupling between source and display.** The source changes independently of the display cycle. The solution must decide when to sample, when to push, and what to show during transition — without the user acting on inconsistent state.
