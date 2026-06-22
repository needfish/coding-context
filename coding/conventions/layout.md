# Code Layout

## Heading Format

```
# ------------------------------------------------------------------------------
#  FILE_NAME.py
#  One-line description.
# ------------------------------------------------------------------------------

# ------------------------------------------------------------------------------
#  SECTION NAME (h0 - file banner)
# ------------------------------------------------------------------------------
# - Section (h1) -
## - Sub-section (h2) -
### - Sub-sub-section (h3) -
```

- h0: file banner. Signals a cohesive unit not meant to be split.
- h1: major feature or module boundary.
- h2: component or logical group within a feature.
- h3: specific function group or sub-component.
- Blank line before and after each header. Uppercase h1--h2, sentence case h3.
- Cap nesting at 3 levels. Evaluate splitting when many h1s appear.

## File Naming

Follow language conventions; otherwise kebab-case for everything. Exceptions: `Dockerfile`, `Makefile`, `.env.example`, `AGENTS.md`. Types kebab-case noun phrases, functions kebab-case verb phrases, variables kebab-case noun phrases, constants `UPPER_SNAKE_CASE`. Private identifiers use `_leading_underscore`.
