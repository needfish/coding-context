# Optionality

A value may be absent. How that absence is modeled determines what the type system enforces, what callers must handle, and what bugs become impossible.

## The Options

| Option | Description | Use when |
|---|---|---|
| Option / Maybe type | Sum type: Some(value) | None. Compiler forces handling both variants. | Absence is a normal case, callers must handle it, compiler enforcement wanted |
| Null / undefined | Language primitive meaning "no value." Checked at runtime. | Language tracks null through its type system (TypeScript strict, Python with basedpyright strict) |
| Empty collection | [] / "" / {} / new Map() | Field is naturally a collection and zero items is a meaningful state |
| Sentinel value | Special instance of the value type marks absence: -1 for index, "" for name | Language has no Option type and no null tracking, or constrained by wire format |
| Separate flag | Boolean field tracks whether the value is present | Constrained by wire format or legacy schema that cannot represent absence directly |
| Separate record types | Two types: one with the field, one without. | Absence of a field changes the structure or behavior of the whole record significantly |

### Option / Maybe type

```typescript
type Option<T> = { kind: "some"; value: T } | { kind: "none" };
```
```rust
fn find_user(id: &str) -> Option<User> { ... }
```
Callers must unwrap or pattern-match at every use site.

### Null / undefined

```typescript
let user: User | null = null;
```
```python
user: User | None = None
```
Every use site must check; a forgotten check is a runtime crash.

### Empty collection

```typescript
let tags: string[] = [];       // no tags
let bio: string = "";          // no bio
let settings: Map<string, Config> = new Map();
```
Preferred over Option<Vec<T>> -- an absent list and an empty list are almost always the same thing. Cannot distinguish "no value" from "empty value."

### Sentinel value

```typescript
let timeout_ms: number = -1;   // -1 means "no timeout"
```
Sentinels are invisible to the type system. Document the sentinel and guard every boundary.

### Separate flag

```typescript
interface Config {
    hasTimeout: boolean;
    timeoutMs: number;  // valid only when hasTimeout is true
}
```
The flag and value can desync. Prefer Option/Maybe over this in application code.

### Separate record types

```typescript
interface UserWithBio {
    name: string;
    bio: string;
}
interface UserWithoutBio {
    name: string;
}
```
Duplicates the shared fields. Extract common fields into a base type or use a discriminated union.

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

- Option<Vec<T>>: an absent list and an empty list are the same thing. Use Vec<T> defaulting to [].
- Null as a third state: T | null | undefined has three states -- avoid this.
- Null in a collection: [1, null, 3] is a different problem from an optional field.
- Forgotten sentinel: callers forget to check sentinel values. Guard every boundary with a validation function.
- Flag/value desync: the boolean says present but the value is uninitialized.

## Invariants

- A field should have at most one absent representation.
- Every boundary that produces or consumes optional values must document what represents absence.
- If a function returns an optional value, every caller must handle the absent case.
- Converting between representations (e.g. null -> Option -> sentinel) must happen at the module boundary, not scattered through business logic.

## See Also

- [invariants.md](invariants.md)
- [application-patterns/neptune-transformation](../application-patterns/neptune-transformation)
