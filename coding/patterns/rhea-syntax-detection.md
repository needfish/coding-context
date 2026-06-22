# ------------------------------------------------------------------------------
#  rhea-syntax-detection.md
#  Behavior variation -- the detection rule mapping changes per file type heuristic.
# ------------------------------------------------------------------------------

# - Problem -

Given a file path, extension, shebang line, or content signature, determine what language or format the file contains so the right processing rules can be applied.

# - When You'll See This -

Any tool that needs to decide how to process a file based on its content type -- syntax highlighting, linting, formatting, diff generation. Detection must work from partial information (just the filename, or just the first line) and handle ambiguous cases gracefully.

# - How It Works -

- **File path**: the path or filename to classify -- may include extension, basename, or full path.
- **Shebang line**: the first line of the file, like `#!/usr/bin/env python3`.
- **Content signature**: magic bytes at the start of the file that identify the format (PDF starts with `%PDF`, PNG with `\x89PNG`).
- **Heuristic rules**: the mapping from signals (extension, shebang, magic bytes) to language identifiers.
- **Language identifier**: a string or enum that names the detected language (`python`, `rust`, `javascript`).

# - What Must Always Be True -

- Every input produces a language identifier -- there's always a fallback (plain text).
- Detection based on extension is instant (no file reads).
- Shebang detection reads at most the first line.
- Magic byte detection doesn't read more bytes than necessary for known signatures.
- When multiple signals conflict, the most specific one wins.

# - What Goes Wrong -

- **Ambiguous extension**: `.md` could be Markdown or Mandoc -- which one do you pick?
- **Wrong shebang**: a file has `#!/usr/bin/python` but contains Rust code -- the shebang lies.
- **Missing extension**: a file with no extension falls back to plain text even when it's valid Python.
- **Performance**: reading content for every file in a large project slows down startup.
- **Heuristic conflicts**: extension says JSON, shebang says nothing, content looks like XML -- what do you report?

# - Design Trade-Offs -

- **Speed vs. accuracy**: extension-based detection is instant but wrong for unusual files; content-based detection is accurate but requires reading the file.
- **Simple vs. comprehensive**: a few rules (extension map) cover 90% of cases; a full detection system (shebang, magic bytes, content analysis) covers 99% but is more code.

# - Related Problems -

- **titan/syntax-highlighting**: detection selects the grammar for highlighting.
- **pluto/input-parsing**: detection selects the format parser.

# - Getting Started -

A hash map from file extension to language identifier: `{".py": "python", ".rs": "rust", ".js": "javascript"}`. If not found, return "text" (plain text). No shebang, no magic bytes, no content analysis.

# - When to Split Into Its Own Module -

- You need shebang detection (files without extensions like CLI scripts).
- Users report wrong detection for common file types.
- You need content-based detection for formats without standard extensions.
- Detection logic needs independent testing from the tools that use it.

# - Signs You're in This Pattern -

- Your code checks file extensions to decide how to process a file.
- You have a list of extension-to-language mappings.
- A file without a recognised extension gets treated as plain text and looks wrong.
- You read the first line of a file to see if it has a shebang.

# - Common Mistakes -

- Reading the whole file to detect the format when the first few bytes suffice.
- Ordering extension checks so that `.js` matches before `.jsx` (specific before general).
- Not providing a fallback language for unrecognised extensions.

# - The Core Challenge -

Detection works with incomplete information. A filename alone doesn't guarantee the content, and reading content to confirm defeats the purpose of fast detection. The challenge is getting the right answer most of the time from minimal signals and failing gracefully when unsure.
