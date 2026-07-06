# ganymede -- output formatting

## Problem

Structured data must be converted to a text format for display, export, or interchange. The format changes per use case: JSON for API responses, CSV for spreadsheet import, HTML for web pages, plain text for terminal output, markdown for documentation.

## Input

- Structured data (list of records, a single object, a stream of items)
- A format specification (what fields to include, date format, locale, null handling)

## Output

- A string in the requested format

## Observable Behavior

- The same data and format always produce the same string
- The formatter does not modify the input data
- Formatting is lossless: parsing the output back gives the original data (unless loss is intended, e.g. rounding)
- Null or missing fields are handled according to the format spec (omit, show "null", show "N/A")

## Minimal Code

```python
import json

def format_json(data) -> str:
    return json.dumps(data, indent=2)

def format_csv(rows: list[dict]) -> str:
    if not rows:
        return ""
    keys = rows[0].keys()
    lines = [",".join(keys)]
    for row in rows:
        lines.append(",".join(str(row.get(k, "")) for k in keys))
    return "\n".join(lines)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Format-specific data loss | CSV output flattens nested structures, losing fields | Document what is lost, or reject formats that cannot represent the data |
| Encoding mismatch | Characters appear garbled (mojibake) in output | Always specify the output encoding; convert explicitly |
| Invalid output format | Produced string is not valid in the target format (malformed JSON, broken CSV with commas in values) | Use a format library (json.dumps, csv.writer) instead of hand-rolling |
| Very large output | Building the entire output string in memory exhausts memory | Stream output: write incrementally to a file or socket |
| Locale-dependent output | Numbers formatted as "1.000,50" in one locale and "1,000.50" in another | Always use a fixed locale (C/POSIX) for machine-readable formats; use user locale only for display |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Library vs. hand-rolled | Library handles edge cases (escaping, quoting, encoding) correctly. Hand-rolled is simpler for tiny formats but easy to get wrong. |
| Eager vs. streaming | Eager builds the full string in memory, simple but may exhaust memory. Streaming writes output piece by piece, handling any size. |
| Machine-readable vs. human-readable | Machine-readable (JSON, protobuf) is strict and parseable. Human-readable (plain text, markdown) is flexible but harder to parse. |

## Start With

Use the standard library formatter for the target format (json.dumps, csv.writer, str for plain text). Add options (null handling, date format, indentation) when needed.

## Split Into Its Own Module When

- More than one output format must be supported.
- Output formatting has configuration options (date format, locale, null handling, indentation).
- The same format is used in multiple places.
- Output needs to be streamed (large datasets).
- Validation of output correctness is needed.

## Signs This Is Present

- Data is converted to a string for display or export.
- json.dumps, csv.writer, str.format, f-strings, or template engines are used.
- API responses are built from domain objects.
- Output options (format, indentation, locale) are configurable.

## Common Mistakes

- Hand-rolling format serialisation instead of using a library.
- Not escaping special characters (commas in CSV, quotes in JSON).
- Assuming all data fits in memory (stream large output).
- Mixing formatting logic with display logic.
- Not testing the round-trip (format + parse gives back the original).
- Hard-coding formatting choices (date format, locale) that should be configurable.
