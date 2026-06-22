# ------------------------------------------------------------------------------
#  type-precision.md
#  Choosing precise types over primitives for domain concepts.
# ------------------------------------------------------------------------------

A domain concept flattened into a primitive (`string`, `int`, `bool`) loses
information. The type system can no longer distinguish a `UserId` from a
`PostId`, a `TemperatureCelsius` from a `SpeedKmh`, or a `Status` with three
valid values from any arbitrary string. You push validation that could be
compile-time to runtime.

## The Principle

**Prefer algebraic data types (ADTs) over primitives.** Two kinds:

- **Product type**: combines several fields -- a record, struct, tuple.
  Means "all of these." Example: `User { name: string, age: int }`.
- **Sum type**: enumerates variants -- a discriminated union, enum.
  Means "one of these." Example: `Status = Pending | Active | Archived`.

Default to ADTs when modeling domain types. They turn ad-hoc representations
(a raw string with a comment listing valid values) into explicit type
definitions the compiler can check.

## The Antipattern: Stringly-Typed Fields

Every time you write `field: string` or `field: int` in a schema, ask:
"Does every value of this primitive make sense here, or only a subset?"

If only a subset, use one of:

| Instead of | Use |
|---|---|
| `status: string` | literal union: `"pending" \| "active" \| "archived"` |
| `role: string` | enum: `enum Role { Admin, Editor, Viewer }` |
| `userId: string` | branded type: `type UserId = string & { __brand: "UserId" }` |
| `temperature: number` | newtype: `struct TemperatureCelsius(f64)` |
| `config: dict` | product type: `struct Config { host: string, port: u16 }` |

## Mechanisms

### Literal unions (TypeScript, Python with Literal)

```typescript
type Status = "pending" | "active" | "archived";
// Any string other than these three is a compile error
```

### Enums (most languages)

```rust
enum Status { Pending, Active, Archived }
// match forces handling of every variant
```

### Branded types (TypeScript)

TypeScript's structural type system treats `string` and `string` as the same
type. A brand adds a phantom property to make them distinct:

```typescript
type Brand<T, B> = T & { __brand: B };
type UserId = Brand<string, "UserId">;
type PostId = Brand<string, "PostId">;

function getUser(id: UserId): User { ... }
// getUser(postId)  -- type error, even though both are strings
```

At runtime the brand disappears -- zero cost. See
[Learning TypeScript -- Branded Types](https://www.learningtypescript.com/articles/branded-types)
and the TypeScript section of [refactoring-catalog.md](../refactoring/refactoring-catalog.md)
for defunctionalization as an alternative pattern.

### Newtypes (Rust)

```rust
struct UserId(String);   // wraps String, distinct from String
struct PostId(String);   // separate type, not interchangeable
```

### Value objects (any language)

A small object that wraps a primitive and enforces invariants in its
constructor:

```python
@dataclass(frozen=True)
class Email:
    value: str

    def __init__(self, value: str):
        if "@" not in value:
            raise ValueError(f"invalid email: {value}")
        object.__setattr__(self, "value", value)
```

## Decision Flow

```
Does every value of this primitive make sense here?
  YES -> keep the primitive (e.g. user's display name is truly free text)
  NO  -> Is the set of valid values fixed and finite?
           YES -> literal union or enum
           NO  -> Does the value carry a domain invariant (format, range, brand)?
                    YES -> branded type / newtype / value object
                    NO  -> keep the primitive (revisit if invariants emerge)
```

## Common Mistakes

- **Branding everything**: not every `string` needs a brand. A `displayName`
  is truly free text. Brand only when swapping values of the same primitive
  type would cause a bug.
- **Enum when the set is open**: if new variants appear without code changes
  (user-defined roles, plugin hook names), an enum forces recompilation.
  Use a string with validation instead.
- **Value object without invariants**: wrapping `int` in `struct Age(int)`
  adds no value unless the constructor enforces `>= 0`.
- **Premature abstraction**: a single `Email` value object is fine. Building
  a `ValueObject` base class with reflection-based equality before you have
  three value objects is not.

## Invariants

- Every domain type that accepts only a subset of a primitive's values must
  be modeled as a distinct type, not documented in a comment.
- At module boundaries, convert to/from primitives. Inside the module, use
  the precise type.
- Branded types and newtypes must be validated at construction time -- the
  brand is a promise that the value has passed validation.

## See Also

- [invariants.md](invariants.md) -- encoding strategy
  (compile-time types top the preference chain)
- [optionality.md](optionality.md) -- modeling absent/optional values
- [many-forms.md](many-forms.md) -- choosing between data representations
- [refactoring-catalog.md](../refactoring/refactoring-catalog.md) -- Opaque
  Type Naming refactoring, defunctionalization
