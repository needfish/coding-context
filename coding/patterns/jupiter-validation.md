# jupiter -- validation

## Problem

Data coming from outside the program (web form, API request, config file, imported file) must be checked against rules. Rules may check: field is present, value is in range, string matches a pattern, or multiple fields relate correctly. If any rule is violated, return a list of errors.

## Input

- Raw data: a dictionary or untyped object
- A schema or set of rules describing valid data

## Output

- Either a validated data structure (if all rules pass)
- Or a list of errors, each with a field name and message

## Observable Behavior

- Data that passes all rules enters the core logic; data that fails any rule is rejected
- The same input and rules always produce the same result
- The validator does not change the input data
- The output type is different from the input type (validated vs. unvalidated)

## Minimal Code

```python
from dataclasses import dataclass
from typing import List

@dataclass
class ValidationError:
    field: str
    message: str

def validate_user(data: dict) -> List[ValidationError]:
    errors = []
    if "name" not in data or not data["name"].strip():
        errors.append(ValidationError("name", "Name is required"))
    if "age" in data:
        try:
            age = int(data["age"])
            if age < 0 or age > 150:
                errors.append(ValidationError("age", "Age must be 0-150"))
        except (ValueError, TypeError):
            errors.append(ValidationError("age", "Age must be a number"))
    return errors
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Confusing parsing with validation | Code rejects structurally valid data because it checks meaning in the parsing step | Keep parsing (structure) and validation (meaning) separate |
| Missing validation | Invalid data reaches core logic and causes errors deep in the program | Validate at every boundary where external data enters |
| Bypassed validation | Frontend validates but backend does not; API calls bypass checks | Always validate in the backend |
| Inconsistent error format | Some errors say "X required", others say "X must not be empty" | Use one standard error format everywhere |
| Expensive validation | Each request makes a database call or HTTP request just to validate | Split fast checks (type, range) from slow checks (lookups) |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Declarative vs. imperative | Declarative (Zod, Pydantic, JSON Schema) is concise and machine-readable. Imperative code can express any rule but is harder to audit. |
| Fail-fast vs. collect-all-errors | Fail-fast stops at first error, simpler but may hide other errors. Collect-all validates all rules and returns every violation. |
| Compile-time vs. runtime | Compile-time (branded types, literal unions) catches errors before running but can't express runtime constraints. Runtime catches everything but needs tests. |

## Start With

The minimal code above using if/elif chains or a schema library (Zod, Pydantic). Use collect-all-errors to give complete feedback.

## Split Into Its Own Module When

- Validation rules are shared across multiple endpoints or input types.
- Validation logic exceeds 50 lines per input type.
- Custom rules (cross-field, database lookups) are needed.
- Validation error formats need to be consistent across the application.

## Signs This Is Present

- if/elif chains check field types, ranges, and formats.
- A schema library (Zod, Yup, Pydantic, Marshmallow) is used.
- Input data is sanitised, normalised, or coerced.
- API responses include error details per field.

## Common Mistakes

- Mixing validation with business logic.
- Using exceptions for validation control flow (return errors as values instead).
- Not validating at internal boundaries.
- Validating the same data multiple times (validate once at entry, pass the validated type internally).
- Assuming a field that passed validation is still valid later (concurrent modification).
