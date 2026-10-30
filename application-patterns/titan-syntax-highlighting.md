# titan -- syntax highlighting

## Problem

Source code must be displayed with different colours or styles for different parts: keywords, strings, comments, types, and function names each get a different visual treatment.

## Input

- Source code text (string)
- A grammar: set of token patterns for the language (keyword list, string delimiters, comment markers)

## Output

- The same source text with colour/style annotations per character range

## Observable Behavior

- Every character in the source is assigned to exactly one token type
- The original text is preserved exactly (only visual styles change)
- The same source and grammar always produce the same highlighting
- The lexer handles all valid syntax of the language
- The lexer does not crash on invalid or malformed code
- Text inside strings and comments is not highlighted as code

## Minimal Code

```python
import re

TOKENS = [
    ("keyword", r"\b(def|class|if|else|return|import|from)\b"),
    ("string",  r'"[^"]*"'),
    ("comment", r"#.*$"),
    ("number",  r"\b\d+\b"),
    ("id",      r"\b[a-zA-Z_]\w*\b"),
]

def lex(source):
    tokens = []
    pos = 0
    while pos < len(source):
        for kind, pattern in TOKENS:
            m = re.match(pattern, source[pos:])
            if m:
                tokens.append((kind, m.group(), pos))
                pos += m.end()
                break
        else:
            pos += 1
    return tokens

def highlight(source):
    tokens = lex(source)
    result = []
    prev_end = 0
    for kind, text, pos in tokens:
        result.append(source[prev_end:pos])
        fmt = {"keyword": "33", "string": "32", "comment": "90", "number": "35"}.get(kind, "0")
        result.append(f"\033[{fmt}m{text}\033[0m")
        prev_end = pos + len(text)
    result.append(source[prev_end:])
    return "".join(result)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Catastrophic backtracking | Some inputs take minutes to highlight | Use a linear-time lexer (state machine, not regex) |
| Incomplete grammar | Language features like template literals, heredocs, raw strings are not highlighted | Expand grammar based on user reports and test with real-world code |
| False highlighting | Words inside strings are highlighted as keywords | Track string/comment state in the lexer |
| Nested languages | HTML with embedded CSS and JS; JS with embedded HTML (JSX) | Use a multi-mode lexer that switches grammar based on context |
| Large file freeze | A 10,000-line file freezes the UI on open | Highlight incrementally (only visible lines) or use async highlighting |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Regex vs. state machine lexer | Regex is concise but risks backtracking and cannot handle nested state. State machine is more verbose but guarantees linear time and handles nesting. |
| Single-pass vs. multi-pass | Single-pass highlights as it scans. Multi-pass (parse -> AST -> highlight) handles complex languages but uses more memory and time. |
| Full vs. incremental | Full highlights the entire file, simple but wastes work on invisible lines. Incremental highlights only visible lines, efficient but needs change tracking. |

## Start With

Use a state machine lexer (or a linear-time lexer generator like tree-sitter). Start with the most common token types for the language. Add string/comment tracking early to prevent false highlighting.

## Split Into Its Own Module When

- More than one language grammar is supported.
- The grammar for a single language exceeds 50 token patterns.
- User-defined highlighting themes or colour schemes are needed.
- Incremental highlighting is required for performance.
- Nested language highlighting is needed.

## Signs This Is Present

- Source code is displayed with coloured keywords, strings, and comments.
- A TextMate or Sublime grammar file is used.
- A tree-sitter grammar is used for syntax-aware highlighting.
- The editor or viewer has a "syntax highlighting" toggle.

## Common Mistakes

- Using a generic regex lexer for all languages (each language has unique lexical rules).
- Not testing with real-world code from large projects.
- Highlighting inside strings and comments.
- Performance issues on large files (highlight only visible lines).
- Not supporting embedded languages (JS inside HTML, CSS inside JSX).
- Using a single colour for all tokens.
