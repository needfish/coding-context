# rhea: Syntax Detection

> **Pressure class:** Behavior variation — the detection rule mapping changes per file type heuristic.

## Problem

A file path, extension, shebang line, or content signature must be mapped to a language identifier so that the correct processing rules (grammar, formatter, linter) can be applied.

## Context

Arises whenever a tool needs to decide how to process a file based on its content type — syntax highlighting (titan), linting, formatting, diff generation. The detection must work from partial information (the file path alone, or just the first line of content) and must handle ambiguous or unknown cases gracefully.

## Domains Involved

- **File path** (lexical): the path or filename of the file to classify. May include extension, basename, or full path.
- **Content sample** (lexical): initial bytes or lines of the file content, typically the shebang line.
- **Mapping rules** (lexical): the set of rules that associate extensions, filenames, shebangs, and content patterns to language identifiers. May include overrides and user-defined mappings.
- **Language identifier** (lexical): the canonical name of the detected language (e.g., "Rust", "Python", "JavaScript").

## Shared Phenomena

- File path → Detector: extension, basename, parent directory names
- Content sample → Detector: shebang line (`#!/usr/bin/env python`), XML declaration (`<?xml`), magic bytes (`%PDF`)
- Mapping rules → Detector: priority-ordered mapping table
- Detector → Consumer: language identifier, confidence score (matched vs. inferred vs. guessed)

## Invariants

- The same file path must always map to the same language identifier, unless mapping rules change.
- A language identifier must be returned for every input — "Unknown" is a valid result, not an error.
- Explicit user overrides (e.g., `-l rust`) must take precedence over automatic detection.
- Mapping by extension must not match files whose extension is ambiguous (e.g., `.h` could be C, C++, or Objective-C) without consulting additional signals.

## Known Failure Modes

- **Extension collision**: The same extension is used by multiple languages (`.h`, `.pl`, `.cl`), and the detector picks the wrong one.
- **Shebang not present**: A script file without a shebang is classified by extension alone, which may be missing or misleading.
- **Shebang override**: An interpreter path in the shebang is non-standard (`#!/usr/local/custom/python3`), and the detector does not recognise it.
- **Content not yet available**: The file content cannot be read (stdin, pipe, remote), so only the filename can be used.
- **User override misspelled**: The user specifies `-l rustt`, which does not match any known language, so the override is silently ignored.
- **Ambiguous with no tiebreaker**: Two mappings match with equal priority and the detector picks one arbitrarily.

## Forces

- **Precision vs. recall**: A conservative detector returns "Unknown" for ambiguous cases (precise but frustrating); an aggressive detector guesses the most likely language (convenient but sometimes wrong).
- **Extension vs. content**: Extension-based detection is fast but fallible; content-based detection is more accurate but requires reading the file, which may be costly or impossible.
- **Built-in vs. configurable rules**: Built-in rules cover common cases; user-configurable rules handle project-specific conventions but add complexity.
- **Single vs. multiple signals**: Using a single signal (extension only) is simple but inaccurate; combining multiple signals (extension + shebang + content + directory context) is more accurate but has more failure modes.

## Related Subproblems

- **titan/syntax-highlight**: Syntax highlighting depends on syntax detection to select the grammar.
- **jupiter/validate**: The detected language may need validation against allowed languages.
- **neptune/transform**: Transformation may depend on the detected language.
- **europa/config-assembly**: Language overrides are part of the configuration.

### ==== Practical Use ====

#### When this pattern matters
Any project that needs to decide how to process a file based on its content type — syntax highlighting, linting, formatting, diff generation.

#### Start with (minimal form)
A hash map from file extension to language name. Python: `{'py': 'python', 'rs': 'rust', 'js': 'javascript'}`. No shebang parsing, no content sniffing.

#### Split into its own module when (observable triggers)
- Extension-based mapping is insufficient — you need shebang detection, content sniffing, or magic bytes.
- Over 50+ language mappings exist — the table becomes unwieldy to maintain in a single file.
- Users need custom overrides for project-specific conventions.
- Detection needs to work from partial information (stdin with no filename).

#### Common over-engineering traps
- Implementing content sniffing before confirming that extension-based detection misses too many files.
- Building a cascading detection pipeline (extension → shebang → content → model) when a single fallback suffices.
- Supporting every language known to exist when your users only use 10.

## Frame Concern

**Information deficit at decision time.** The detector must commit to a language before it has all the information — the file path is available immediately, but the content (shebang, magic bytes) may require I/O. The concern is how to make the best possible decision with whatever information is currently available, and whether to revise the decision when more information arrives.
