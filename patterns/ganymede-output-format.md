# ganymede: Output Formatting

## Problem

Structured data must be rendered as human-readable text, applying contextual styling (colors, icons, indentation) based on data attributes, terminal capabilities, and user preferences.

## Context

Arises wherever a command-line tool displays results to a terminal — search results, file listings, status reports, diff output. The formatting is the final step before output; it consumes structured entries and produces formatted strings. It may or may not involve colour, depending on terminal support and configuration.

## Domains Involved

- **Entries** (lexical): structured data items to be formatted. Each has typed fields (path, type, size, permissions, timestamps).
- **Format template** (lexical): the specification of how each entry is rendered. May include field order, separators, conditional elements.
- **Style rules** (lexical): the mapping from entry attributes to visual styles (colours, fonts, icons). Typically comes from environment variables (`LS_COLORS`) or configuration files.
- **Output stream** (causal): the destination for formatted text — terminal, pipe, file. Determines whether styling is applied.

## Shared Phenomena

- Entries → Formatter: attribute values, metadata
- Format template → Formatter: format string, field specifications
- Style rules → Formatter: colour codes, icon characters, font effects
- Output stream → Formatter: terminal width, colour support, encoding
- Formatter → Output: formatted string

## Invariants

- Formatting must not lose or alter data — it transforms representation, not content.
- Styling (colours, icons) must be applied only when the output stream supports it.
- Formatting must handle entries with missing fields gracefully (use default or placeholder).
- The formatted output for the same entry and same format must be identical across invocations.

## Known Failure Modes

- **Garbled output to pipe**: Escape codes intended for a terminal are piped into a file or another command, producing visible garbage.
- **Encoding mismatch**: The formatted string contains characters that the output stream encoding cannot represent.
- **Width overflow**: The formatted line exceeds the terminal width, causing wrapping or truncation that breaks alignment.
- **Missing style source**: The style rules reference colours or icons that are not available (missing environment variable, unsupported terminal).
- **Performance**: Formatting a large number of entries with expensive styling (e.g., spawning `ls` for each entry) blocks output.
- **Inconsistent alignment**: Variable-width characters (Unicode, CJK) break column alignment when the formatter assumes fixed-width cells.

## Forces

- **Richness vs. portability**: Colourful, icon-rich output is informative but only works in supporting terminals; plain text works everywhere.
- **Breadth vs. depth**: A single line per entry (list format) is compact; multi-line per entry (detail format) is informative but verbose.
- **Default vs. customisable**: Sensible defaults work for most users; customisable format templates add complexity but cover edge cases.
- **Pre-computation vs. lazy rendering**: Pre-computing formatted strings batches work but uses memory; rendering on the fly saves memory but may block between entries.

## Related Subproblems

- **mercury/display**: Output formatting feeds display.
- **neptune/transform**: Entries may need transformation before formatting.
- **venus/filter**: Only filtered entries need formatting.
- **earth/sort**: Sorted entries are formatted in order.
- **callisto/walk**: File system entries are the most common input to formatting.

## Frame Concern

**Terminal capability detection.** The formatter must decide whether to apply styling before it knows if the output is going to a terminal, a pipe, or a file — and the terminal may support different colour depths, widths, and Unicode ranges. The concern is making the styling decision at the right time with incomplete information about the output path.
