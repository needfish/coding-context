# Error Handling

- Validate at function entry and fail fast. Parse into valid forms or reject.
- Use the language's standard error mechanism.
- Log at the detection point, recover at the decision point.
- Wrap external errors: `raise MyDomainError from original_exc`.
- Prefer early returns over deep nesting.
