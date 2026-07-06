# rhea -- syntax detection

## Problem

Given a file or data stream without explicit format information (or with ambiguous information), determine what format it uses. The detector uses the filename, the first few bytes (magic bytes), the shebang line, and content patterns to make a guess.

## Input

- A filename (may have an extension)
- The first N bytes of the file (typically 4KB)
- Optionally, the first line (shebang)

## Output

- A format name (string): "python", "json", "png", "markdown", etc.
- Or "unknown" if no format matches

## Observable Behavior

- Every input produces a result (never crashes)
- The same input always produces the same result
- The detector uses a priority order: magic bytes > shebang > extension > content patterns
- Only the first N bytes of the file are read (not the entire file)
- The detector returns "unknown" when no format matches with sufficient confidence

## Minimal Code

```python
import os

MAGIC_BYTES = {
    b"\x89PNG": "png",
    b"\xff\xd8\xff": "jpeg",
    b"%PDF": "pdf",
}

def detect_format(path):
    ext = os.path.splitext(path)[1].lower()
    if ext in {".py", ".js", ".ts", ".rs", ".go", ".c", ".h", ".cpp"}:
        return ext[1:]

    with open(path, "rb") as f:
        header = f.read(4)

    for magic, fmt in MAGIC_BYTES.items():
        if header.startswith(magic):
            return fmt

    return "unknown"
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Ambiguous extension | .md could be Markdown or Mandoc | Check content patterns (headings, HTML tags, code fences) |
| Heuristic conflict | Extension says JSON, content looks like XML | Define a priority order for detection signals |
| False positive | Plain text detected as Python because it contains "def " on the first line | Combine multiple signals (shebang + keywords + indentation) |
| False negative | Valid Ruby file detected as "unknown" because it lacks a shebang | Expand the format registry with more content patterns |
| Large file performance | Detecting format by scanning entire file is slow | Only scan the first N bytes (typically 4KB) |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Accuracy vs. speed | Deeper analysis (parse first few lines, check grammar) improves accuracy but takes time. Shallow analysis (extension + magic bytes) is fast but less accurate. |
| Specificity vs. generality | Recognising 200+ formats can distinguish closely related ones but may have more false positives. Recognising 10 formats is more reliable for those 10 but fails on everything else. |
| Extension-based vs. content-based | Extension is instant but unreliable. Content (magic bytes, shebang, patterns) is more reliable but requires scanning. |

## Start With

Check the file extension first, then magic bytes, then content patterns. Only scan the first 4KB. Add rules for new formats as they appear.

## Split Into Its Own Module When

- More than 5 formats need to be detected.
- Detection needs to be customised by the user (associate extensions with formats).
- Detection needs to support format variants (Markdown vs. Mandoc for .md).
- The detection logic is used by multiple consumers (highlighter, parser, file browser).

## Signs This Is Present

- A file extension is used to decide how to process a file.
- Magic bytes or shebang lines are checked.
- A "detect language" or "detect format" function exists.
- An if/elif chain checks extension -> content -> fallback.

## Common Mistakes

- Relying only on file extension for security-critical decisions (a .txt file can contain JavaScript).
- Not normalising the extension before comparison (case, leading dot).
- Scanning the entire file instead of the first N bytes.
- Not handling binary files in content-based detection.
- Using a heuristic that has false positives for common files.
- Not caching the detection result when the same file is probed multiple times.
