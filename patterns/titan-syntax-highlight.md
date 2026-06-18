# titan: Syntax Highlighting

> **Pressure class:** Behavior variation — the grammar-driven classification algorithm changes per language.

## Problem

Source code or structured text must be classified by language and annotated with semantic styles (keywords, strings, comments, etc.) to improve human readability.

## Context

Arises in any tool that displays source code — text editors, pagers, diff viewers, code review systems, documentation generators. The highlighting is a presentation transform: it does not change the content, only its visual rendering. It sits between input reading (input/reading) and output formatting (ganymede), and depends on a language classification step (syntax detection).

## Domains Involved

- **Source text** (lexical): the raw character stream to be highlighted.
- **Syntax grammar** (lexical): the set of rules defining token types for a particular language — keywords, operators, string delimiters, comment markers, etc. Typically comes from a grammar library (Sublime Text `.sublime-syntax` files, TextMate `.tmLanguage`, LSP grammar definitions).
- **Color theme** (lexical): the mapping from token types to visual styles (colour, bold, italic, underline). Typically comes from a theme file (`.tmTheme`, `.json`).
- **Token stream** (lexical): the annotated sequence of tokens produced by the grammar engine.

## Shared Phenomena

- Source text → Highlighter: character stream, byte offsets, line boundaries
- Syntax grammar → Highlighter: token patterns, scope hierarchy, embedding rules
- Highlighter → Token stream: (token type, text range, scope) tuples
- Color theme → Token stream: (style, token type) mapping
- Token stream → Output: styled spans of text with applied colours

## Invariants

- Highlighting must not alter the source text content — it only adds visual annotations.
- Every character must be assigned to exactly one token type (no gaps, no overlaps).
- A token type must have a defined style in the theme, or fall back to a default (usually the "text" or "default" scope).
- The highlighting for a given source text and grammar must be deterministic.
- Embedded languages (e.g., JavaScript inside HTML, SQL inside Python) must be handled by the grammar engine's nesting rules.

## Known Failure Modes

- **Misclassification**: The wrong grammar is applied (e.g., highlighting a Markdown file as C++), producing nonsensical colours.
- **Incomplete grammar**: The grammar does not cover all constructs of the language, leaving some text unstyled or mis-styled.
- **Performance**: Re-highlighting the entire file on every keystroke in an editor is too slow; incremental highlighting is needed but complex.
- **Theme gap**: A token type present in the grammar has no corresponding entry in the theme, causing it to render in the default colour (often invisible against the background).
- **Grammar conflict**: Two grammars claim the same file extension, or nested grammars conflict on delimiter boundaries.
- **Encoding mismatch**: The grammar expects a particular encoding (UTF-8) but the source text uses a different encoding, causing garbled tokenization.

## Forces

- **Completeness vs. performance**: A grammar that covers every edge case of a language is large and slow; a simpler grammar covers 95% of real code and runs much faster.
- **Generality vs. specificity**: A generic grammar (keywords, strings, comments) works for most languages but misses language-specific semantics; a custom grammar per language is more accurate but requires maintenance.
- **Synchronous vs. asynchronous**: Synchronous highlighting blocks display until complete (simple); asynchronous highlighting shows partial results (responsive but complex).
- **Whole-file vs. incremental**: Whole-file highlighting is simple but wastes work on unchanged parts; incremental highlighting is efficient but needs to track which lines changed.

## Related Subproblems

- **syntax detection**: Language classification must precede highlighting — the correct grammar must be selected from a path, extension, shebang, or content analysis.
- **mercury/display**: Highlighted text is ultimately displayed.
- **ganymede/output-format**: Styled tokens must be formatted for the output medium (terminal ANSI, HTML, SVG).
- **neptune/transform**: Source text is transformed into a styled representation.
- **venus/filter**: Highlighting may be filtered by line range or visible region.

### ==== Practical Use ====

#### When this pattern matters
Any project that displays source code — text editors, pagers (bat), diff viewers, code review tools, documentation generators.

#### Start with (minimal form)
Use an existing highlighting library (syntect, Pygments, highlight.js, Prism). Feed entire file, get styled output. No incremental update, no custom grammar.

#### Split into its own module when (observable triggers)
- The integrated library doesn't support a language you need — you need custom grammar rules.
- Re-highlighting the entire file on every keystroke is too slow — you need incremental re-highlighting.
- You need to support embedded languages (JS in HTML, SQL in Python) that the library doesn't handle.
- The output target changes (terminal vs HTML vs SVG) and needs different renderers.

#### Common over-engineering traps
- Writing a grammar engine from scratch when an existing library covers your languages.
- Building an incremental highlighter before measuring that full-file re-highlight is too slow.
- Supporting every colour theme format when one format (`.tmTheme`, `.json`) suffices.

## Frame Concern

**Grammar coverage vs. source variety.** The highlighter depends on a grammar library that can never be complete — new languages, new syntax features, and edge cases in existing languages mean the grammar is always a partial approximation. The concern is how the system behaves when encountering source text that its grammar does not fully cover.
