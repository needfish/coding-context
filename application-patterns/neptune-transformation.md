# neptune -- transformation

## Problem

Data in one form must be converted to another form. Examples: XML to JSON, Celsius to Fahrenheit, absolute file paths to relative paths, user IDs to user names. The conversion logic changes per use case but always takes one value and returns another.

## Input

- Data in the source representation (any type: string, dict, number, object)

## Output

- Data in the target representation (possibly a different type)

## Observable Behavior

- Given the same input, the transform always returns the same output
- The transform does not modify the input
- The transform does not read files, make network calls, or change global state
- If a field is missing or null in the input, the transform returns null/None for the corresponding output field (does not crash)

## Minimal Code

```python
def transform_user(source: dict) -> dict:
    return {
        "id": source["user_id"],
        "name": source["full_name"],
        "email": source["email_address"],
        "active": source["status"] == "active",
    }
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Lossy transformation | Nested data flattened to CSV loses some fields | Document what is lost, or provide alternative output format |
| Side effects in transform | Calling the transform writes to a file or makes a network request | Keep transforms pure; do I/O before or after |
| Every field change requires editing every transform | Adding one field to source means updating 10 transform functions | Use schema-driven mapping or default values for unknown fields |
| Encoding mismatch | Characters appear garbled in output | Declare encoding at both boundaries and convert explicitly |
| Batch partial failure | One item fails and all results are lost | Handle each item independently; skip failed items and report them |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Explicit vs. schema-driven mapping | Explicit (field-by-field in code) is type-safe but verbose. Schema-driven (JSON Schema -> JSON Schema) is concise but limited to structural transforms. |
| Eager vs. lazy | Eager produces the full output in memory, simple but may exhaust memory for large datasets. Lazy (generators) processes one item at a time. |
| One-step vs. multi-step | One-step (source -> target) is simple but duplicates logic. Multi-step (source -> intermediate -> target) reuses intermediates. |

## Start With

Write a function that takes the source type and returns the target type. Keep it pure (no side effects, no I/O). Use explicit field-by-field mapping.

## Split Into Its Own Module When

- The same transformation is needed in multiple places.
- Transformation logic is complex enough to require unit tests.
- A data format change requires updating multiple transforms.
- The transformation has configuration options (include nulls, date format, locale).

## Signs This Is Present

- A function takes one data shape and returns another data shape.
- Data is remapped, renamed, reformatted, or converted.
- A dto, view model, or api adapter is defined.
- Format conversion code exists (xml_to_json, markdown_to_html).

## Common Mistakes

- Writing transforms that modify their input.
- Forgetting to handle null/absent fields in the source.
- Copy-pasting the same transform logic across multiple files.
- Doing I/O inside a transform (network call, file read, database query).
- Assuming the source structure is always correct.
- Not testing the round-trip (transform forward then backward gives the original).
