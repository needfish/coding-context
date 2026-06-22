# ------------------------------------------------------------------------------
#  optionality.md
#  Modeling absent, missing, or optional data.
# ------------------------------------------------------------------------------

A value may be absent. How you model that absence determines what the type
system enforces, what callers must handle, and what bugs become impossible.

## The Options

### 1. Option / Maybe type

A sum type: `Some(value) | None`. The compiler forces callers to handle both
variants -- no forgetting to check for null.

```typescript
// TypeScript
type Option<T> = { kind: "some"; value: T } | { kind: "none" };
// Or built-in: T | undefined / T | null (weaker, no pattern-match enforcement)
```

```python
# Python
from dataclasses import dataclass
from typing import Generic, TypeVar, Union

T = TypeVar("T")

@dataclass
class Some(Generic[T]):
    value: T

@dataclass
class None_:
    pass

Option = Union[Some[T], None_]
```

```rust
// Rust (built-in)
fn find_user(id: &str) -> Option<User> {
    // ...
}
```

- Use when: the absence is a normal case, callers must handle it, and you want
  the compiler to enforce that.
- Trade-off: callers must unwrap or pattern-match at every use site.

### 2. Null / undefined

A language primitive that means "no value." Checked at runtime, not at compile
time (unless the type system tracks nullability).

```typescript
// TypeScript with strictNullChecks
let user: User | null = null;
```

```python
# Python
user: User | None = None
```

- Use when: the language tracks null through its type system (TypeScript strict,
  Rust forbids null entirely, Python with basedpyright strict).
- Avoid when: the language does not track null (JavaScript without strict, older
  Python, C).
- Trade-off: every use site must check; a forgotten check is a runtime crash.

### 3. Empty collection

An empty list, set, string, or map represents absence.

```typescript
let tags: string[] = [];       // no tags
let bio: string = "";          // no bio
let settings: Map<string, Config> = new Map();  // no settings
```

- Use when: the field is naturally a collection and zero items is a meaningful
  state. Prefer this over `Option<Vec<T>>` -- an absent list and an empty list
  are almost always the same thing.
- Trade-off: cannot distinguish "no value" from "empty value." When that
  distinction matters, use Option.

### 4. Sentinel value

A special instance of the value type marks absence: `-1` for index, `""` for
name, `MAX_INT` for "no timeout."

```typescript
let timeout_ms: number = -1;   // -1 means "no timeout"
```

- Use when: the language has no Option type and no null tracking, or you are
  constrained by a wire format that has no null representation (URL query
  params, some CSV conventions).
- Trade-off: sentinels are invisible to the type system. Nothing prevents a
  caller from treating `-1` as a real timeout. Document the sentinel and guard
  every boundary.

### 5. Separate flag

A boolean field tracks whether the value is present; the value field is only
valid when the flag is true.

```typescript
interface Config {
    hasTimeout: boolean;
    timeoutMs: number;  // valid only when hasTimeout is true
}
```

- Use when: constrained by a wire format or legacy schema that cannot represent
  absence directly (flat buffers, packed structs).
- Trade-off: the flag and value can desync. Prefer Option/Maybe over this in
  application code. This is an implementation detail, not a modeling choice.

### 6. Separate record types

Two types: one with the field, one without. The variant is chosen at the
record level.

```typescript
interface UserWithBio {
    name: string;
    bio: string;
}
interface UserWithoutBio {
    name: string;
}
```

- Use when: the absence of a field changes the structure or behavior of the
  whole record significantly -- e.g. a draft vs. published post, where
  `publishedPost` has a `publishDate` that `draftPost` lacks.
- Trade-off: duplicates the shared fields. Extract the common fields into a
  base type or use a discriminated union.

## Decision Flow

```
Is the field naturally a collection (list, set, string, map)?
  YES -> empty collection ([] / "" / new Map())
  NO  -> Is absence a normal, expected case?
           YES -> Is the language's null tracking reliable enough?
                    YES -> T | null / T | undefined
                    NO  -> Option/Maybe type
           NO  -> Is the value required?
                    YES -> just T (no optionality)
                    NO  -> Option/Maybe type
```

## Common Mistakes

- **Option<Vec<T>>**: an absent list and an empty list are the same thing in
  almost every domain. Use `Vec<T>` defaulting to `[]`, not `Option<Vec<T>>`.
- **Null as a third state**: a field typed `T | null` has two states (present,
  absent). A field typed `T | null | undefined` has three -- avoid this.
- **Null in a collection**: `[1, null, 3]` is a different problem (sparse data)
  from an optional field. Use a separate absent marker or filter nulls out.
- **Forgotten sentinel**: a sentinel value like `-1` or `""` that callers
  forget to check for. Guard every boundary with a validation function that
  converts sentinels to Option at the module edge.
- **Flag/value desync**: the boolean says present but the value is uninitialized.
  Encode as a sum type so the two states cannot get out of sync.

## Invariants

- A field should have at most one absent representation. Do not mix Option,
  null, and sentinel in the same field.
- Every boundary that produces or consumes optional values must document what
  represents absence.
- If a function returns an optional value, every caller must handle the absent
  case (compiler-enforced with Option/Maybe, convention-enforced with null).
- Converting between representations (e.g. null -> Option -> sentinel) must
  happen at the module boundary, not scattered through business logic.

## See Also

- [invariants.md](invariants.md) -- nullability as a
  domain constraint category
- [patterns/neptune-transformation.md](../patterns/neptune-transformation.md) --
  common mistake: forgetting to handle null/missing fields during transformation
