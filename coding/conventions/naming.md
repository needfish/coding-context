# Naming Conventions

## Style

- Use `kebab-case` (snake_case when `-` is the subtraction operator). Use `ALL_CAPS` for constants.
- Use full words, never abbreviate, use as many words as needed.
- Mirror the type name in the variable name: `user-profile` for a `user-profile` type.

## Grammar Patterns for Identifiers

| Pattern                                 | Use for                | Examples                                    |
|-----------------------------------------|------------------------|---------------------------------------------|
| NM\* N                                  | Types, classes, vars   | `selection-width-label`, `user-name`        |
| NM\* NPL                                | Collections            | `training-examples`, `customers`            |
| V NM\* (N\|NPL)                         | Functions, booleans    | `filter-users-by-name`, `is-active`         |
| P NM\* (N\|NPL)                         | Conversions, events    | `string-from-document`, `on-enter`          |
| NM\* N P NM\* (N\|NPL)                  | Entity relationships   | `timeout-in-milliseconds`, `url-from-json`  |
| V P NM\* (N\|NPL)                       | Actions with target    | `serialize-to-pdf`, `save-as-png`           |
| V\* DT NM\* (N\|NPL)                    | Population ops         | `all-open-indices`, `matches-any-category`  |
| V                                        | Generic actions        | `sort()`, `delete()`                        |

## Antipatterns

- Do not use `is*` for non-boolean returns.
- Do not use a singular name for a collection return.
- `check` must not return void.
- A predicate name must return a boolean.
- A transform must return the transformed value.
- A singular name on a collection type must be pluralized.
- A boolean name on a non-boolean type must be changed to the correct type.
- A plural name on a scalar type must be singularized.

## Types

Use a noun phrase. Name after the data or its state (`validated-order`, not `order-validator`). Avoid `-er` and `-or` suffixes. Do not use `Manager`/`Handler`/`Processor`/`Helper`/`Data`/`Info`. Document each type with what it represents, includes, and excludes.

## Functions

Use a verb phrase in imperative mood (`filter-users`, not `filters-users`). Say what, not how (`filter-users-by-name` not `fetch-and-filter`). For conversions: `as-` (zero-cost, e.g. `as-bytes`), `to-` (expensive, e.g. `to-string`), `from-` (construction, e.g. `json-from-order`). Use `result-from-source` ordering. Lifecycle suffixes: `init`, `release`, `clear`. Booleans read as questions (`is-active`, `has-permission`). Split if the name contains "and" or "or".

## Variables

Collections use plural or collective nouns. Booleans use positive sense. Loop indexes use `i_` suffixed with the collection abbreviation (`i_c`). Use `i`, `j`, `k` only in the tightest scope (three lines or fewer).

## Parameters

Name the role (`name-query`), not the type (`term`).

## Constants

Name what the value represents (`BOILING_POINT`), not the value itself (`ONE_HUNDRED`).

## Vocabulary Rules

- Use dictionary words (`account` not `acc`).
- Scope-appropriate length: `i` in tight scope, `customer-record-count` in broad.
- Write full words. Accept only domain-standard abbreviations: `html`, `http`, `id`, `db`, `str`, `buf`.
- Module prefix: `order-validate` to group all `order-` names together.
- Use the domain's actual term (`tuple` not `row` in parser code).
- Positive booleans: `is-enabled` not `is-not-disabled`.
- Consistent opposite pairs: `first`/`last`, `start`/`stop`.
- Lifecycle verbs: `init`, `release`, `clear`, `free`.

## Decision Flow

```
WHAT KIND OF THING?
|-- TYPE -> kebab-case noun phrase. Domain has a name? Use it. Avoid -er/-or.
|-- FUNCTION -> kebab-case verb phrase. Boolean -> is-/has-/can-. Conversion -> result-from-source.
|-- VARIABLE -> kebab-case noun phrase. Collection -> plural. Boolean -> positive. Loop -> i_<coll>.
L-- PARAMETER -> kebab-case noun phrase. Name the role, not the type.

VERIFY:
  [] kebab-case, full words, as many as needed
  [] No "and"/"or" (split if present)
  [] No -er/-or on types (domain nouns are exceptions)
  [] No Manager/Handler/Processor/Info/Data
  [] Name matches behavior (check antipatterns)
  [] Positive boolean sense, plural iff collection
  [] Consistent opposite pairs
```
