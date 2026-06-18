# iapetus: Code Counting / Statistics Aggregation

## Problem

Text content must be classified into categories (code, comments, blanks) line by line, and counts must be aggregated across files and grouped by language, directory, or other criteria.

## Context

Arises in code analysis tools, project health dashboards, billing systems that charge by lines of code, and report generators. The counting depends on language-specific syntax rules (comment markers, string delimiters, nesting rules) applied at the line level. The aggregation combines individual file counts into group totals (by language, by directory, by contributor).

## Domains Involved

- **Source text** (lexical): the raw text content of each file, split into lines.
- **Language grammar** (lexical): the syntactic rules for the source language — single-line comment markers, multi-line comment delimiters, string literals, doc comment markers.
- **Line classifier** (causal): the state machine that reads lines and assigns each to a category (code, comment, blank, doc comment).
- **Aggregator** (causal): combines per-file counts into group totals.
- **Report** (lexical): the output structure of aggregated counts, typically grouped by language or directory.

## Shared Phenomena

- Source text → Classifier: lines of text
- Grammar → Classifier: comment markers, string delimiters, nesting rules
- Classifier → Aggregator: per-file (code, comment, blank) counts
- Aggregator → Report: group totals, subtotals, hierarchical breakdowns

## Invariants

- Every line must be assigned to exactly one category — no line may be double-counted or unclassified.
- The sum of code + comment + blank lines must equal the total lines in the file.
- The classifier must correctly handle nested comments (e.g., `/* /* */ */`) and comments embedded in string literals.
- Multi-line comments must be tracked across line boundaries — if a comment starts on line N, lines N through M (where M contains the closing delimiter) are all comment, even if they contain code-like text.
- Aggregation must be deterministic: the same set of files and same grammar must produce the same counts.

## Known Failure Modes

- **Edge case in grammar**: A language construct that is not covered by the grammar causes misclassification (e.g., a heredoc containing comment-like text).
- **Nested comment mishandling**: A comment opener inside a comment block is not recognised, causing the classifier to miss the closing delimiter and classify subsequent lines incorrectly.
- **String vs. comment ambiguity**: A string literal contains text that matches a comment opener (e.g., `"/*"`), causing the classifier to incorrectly enter comment mode.
- **Shebang line**: The first line starting with `#!` is classified as code in some languages (e.g., Python) but as a comment in others (e.g., shell).
- **Mixed-language files**: A file contains embedded code in another language (e.g., SQL in Python, JavaScript in HTML), and the classifier only recognises the outer language.
- **Unicode confusion**: Multi-byte characters or zero-width spaces cause the classifier's seek operations to misalign.

## Forces

- **Accuracy vs. speed**: A per-character state machine is accurate but slower; regex-based line matching is fast but misses edge cases like nested comments.
- **Generality vs. specificity**: A generic counting engine works across languages but misses language-specific constructs; per-language grammars are accurate but require maintenance for each language.
- **Line vs. character granularity**: Line-level classification is fast and aligns with display units; character-level is more detailed but slower and rarely needed for statistics.
- **Total vs. incremental counting**: Counting all files from scratch is simple but wasteful; incremental counting updates only changed files but must handle deletions and renames.

## Related Subproblems

- **rhea/syntax-detection**: The correct language grammar must be selected before counting begins.
- **callisto/filesystem-walk**: Files must be discovered before they can be counted.
- **venus/filter**: Only a subset of files may be counted (by extension, by path, by modified date).
- **ganymede/output-format**: The aggregated report must be formatted for display or serialised for export.
- **earth/sort**: Report rows are typically sorted by count or name.
- **mercury/display**: The report is displayed to the user.

## Frame Concern

**Grammar coverage vs. real-world source variety.** A code counter works by classifying text according to language rules, but real-world source code contains edge cases (nested comments in strings, preprocessor directives, template languages embedded in code) that any fixed grammar cannot fully anticipate. The concern is how the classifier behaves when it encounters text that its grammar does not correctly describe — whether it fails safe (misses a count) or fails unsafely (causes a cascade of misclassifications).
