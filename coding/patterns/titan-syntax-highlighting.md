# ------------------------------------------------------------------------------
#  titan-syntax-highlighting.md
#  Behavior variation -- the grammar-driven classification algorithm changes per language.
# ------------------------------------------------------------------------------

# - Problem -

Source code needs to be classified by language and annotated with semantic styles -- keywords in one colour, strings in another, comments in a third -- to make it easier for humans to read.

# - When You'll See This -

Any tool that displays source code -- text editors, pagers, diff viewers, code review systems, documentation generators. Highlighting is a presentation transform: it changes only the visual rendering, not the content. It sits between input reading and output formatting.

# - How It Works -

- **Source text**: the raw character stream to be highlighted.
- **Grammar**: a set of rules that define how to classify each token -- keywords, strings, comments, operators, identifiers.
- **Token stream**: the classified output -- a sequence of tokens, each with a type (keyword, string, comment, etc.) and a position.
- **Style map**: a mapping from token types to visual styles (colour, bold, italic, underline).

# - What Must Always Be True -

- Every character in the source is assigned to exactly one token.
- The highlighted output is visually equivalent to the original for someone who can't see colours.
- The grammar handles nested constructs correctly (strings inside code, comments that look like code).
- The highlighter doesn't modify the source text.

# - What Goes Wrong -

- **Wrong language detected**: a `.js` file highlighted as Python looks wrong and confuses the reader.
- **Nested constructs break**: a string containing a comment marker gets split incorrectly.
- **Performance hang**: highlighting a 100,000-line file freezes the UI.
- **Incomplete grammar**: a new language feature isn't recognised and shows as plain text.
- **Context confusion**: a keyword inside a string is highlighted as a keyword instead of as string content.

# - Design Trade-Offs -

- **Correctness vs. speed**: full parsing gives perfect highlighting but is slow; regex-based highlighting is fast but gets edge cases wrong.
- **Completeness vs. simplicity**: supporting every language is ideal but requires hundreds of grammars; supporting the top 10 covers most use cases.

# - Related Problems -

- **rhea/syntax-detection**: you need to detect the language before you can highlight it.
- **pluto/input-parsing**: parsing the source to understand its structure.
- **ganymede/output-formatting**: highlighted tokens need to be rendered in the output format.

# - Getting Started -

A hash map of keyword colours and a regex for strings and comments. Apply colours sequentially. Python: use `pygments` or a hand-written loop with `re`. No grammar files, no tree-sitter, no incremental parsing.

# - When to Split Into Its Own Module -

- You need to support more than one language (grammar selection needed).
- You need incremental highlighting (re-highlight only the changed part on edit).
- You need to test highlighting independently from the editor/viewer.
- Users report highlighting mistakes in edge cases.

# - Signs You're in This Pattern -

- You're colouring parts of text based on what they mean (keywords, strings, comments).
- Your code has a list of keyword patterns or a grammar definition.
- You need to re-apply colours when the text changes.
- The highlighting logic is mixed with the display logic.

# - Common Mistakes -

- Building a full parser just for highlighting when regex works for 95% of cases.
- Writing grammars for every language before users request them.
- Highlighting synchronously on the UI thread for large files.

# - The Core Challenge -

A highlighter is a parser that's allowed to be wrong. It trades correctness for speed and simplicity. The challenge is knowing which edge cases to get right (string boundaries, nested comments) and which ones to accept as wrong (rare syntax in a language you barely support).
