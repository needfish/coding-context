# ------------------------------------------------------------------------------
#  defunctionalization.md
#  Cross-paradigm transformation: replacing higher-order functions with data
#  types + interpreters, and vice versa.
# ------------------------------------------------------------------------------

**Author:** Jimmy Koppel (MIT, pathsensitive.com)
**Sources:**
- Talk: "The Best Refactoring You've Never Heard Of" -- <https://www.youtube.com/watch?v=vNwukfhsOME>
- Blog post: <https://www.pathsensitive.com/2019/07/the-best-refactoring-youve-never-heard.html>
- SIGPLAN article: <https://blog.sigplan.org/2019/12/30/defunctionalization-everybody-does-it-nobody-talks-about-it/>

This is not a catalog entry but a **primordial transformation** that explains many
seemingly-unrelated refactorings across all paradigms. Understanding
defunctionalization lets you derive design alternatives mechanically rather than
relying on memorized catalog entries.

## What It Is

**Defunctionalization** replaces higher-order functions (lambdas, closures,
callbacks, strategies) with a data type + an interpreter (apply function). Its
inverse, **refunctionalization**, replaces a data type + interpreter with
higher-order functions.

```haskell
-- Before (higher-order, "refunctionalized"):
filter :: (Int -> Bool) -> [Int] -> [Int]
filter isOdd [1,2,3]
filter (\x -> x < y) [1,2,3]

-- After (defunctionalized):
data Filter = IsOdd | LessThan Int | And Filter Filter
apply :: Filter -> Int -> Bool

filter :: Filter -> [Int] -> [Int]
filter IsOdd [1,2,3]
filter (LessThan y) [1,2,3]
```

**Python example -- strategy pattern defunctionalized:**

```python
# Before (higher-order): callers pass a lambda
def process_items(items, *, transform):
    return [transform(x) for x in items]

# Callers:
process_items(data, transform=lambda x: x.upper())
process_items(data, transform=lambda x: x.strip())
process_items(data, transform=lambda x: x[::-1])

# After (defunctionalized): transform becomes a data type + apply function
from dataclasses import dataclass
from typing import Union

@dataclass
class UpperCase:
    pass

@dataclass
class Strip:
    pass

@dataclass
class Reverse:
    pass

Transform = Union[UpperCase, Strip, Reverse]

def apply_transform(t: Transform, value: str) -> str:
    match t:
        case UpperCase():
            return value.upper()
        case Strip():
            return value.strip()
        case Reverse():
            return value[::-1]

def process_items(items, *, transform: Transform):
    return [apply_transform(transform, x) for x in items]

# Callers:
process_items(data, transform=UpperCase())
process_items(data, transform=Strip())
process_items(data, transform=Reverse())

# Benefit: Transform can be serialized, logged, compared, queued
# Cost: adding a new transform requires adding a dataclass + match arm
```

**JavaScript/TypeScript example -- async callback defunctionalized:**

```typescript
// Before (higher-order): callbacks passed to async operations
function fetchData(url: string, onSuccess: (data: any) => void, onError: (err: Error) => void) {
    http.get(url, (res) => {
        let body = "";
        res.on("data", (chunk) => (body += chunk));
        res.on("end", () => onSuccess(JSON.parse(body)));
    }).on("error", onError);
}

// Callers:
fetchData("/api/users", (data) => renderUsers(data), (err) => showError(err));
fetchData("/api/posts", (data) => renderPosts(data), (err) => showError(err));

// After (defunctionalized): each callback pair becomes a message
// (This is what happens under the hood when you use a reducer/state machine)
type Action =
    | { kind: "fetch-users" }
    | { kind: "fetch-posts" }
    | { kind: "fetch-user"; userId: string };

type State = { users: any[]; posts: any[]; error: string | null };

function applyAction(state: State, action: Action): Promise<State> {
    switch (action.kind) {
        case "fetch-users":
            return fetch("/api/users")
                .then((r) => r.json())
                .then((users) => ({ ...state, users }))
                .catch((err) => ({ ...state, error: err.message }));
        case "fetch-posts":
            return fetch("/api/posts")
                .then((r) => r.json())
                .then((posts) => ({ ...state, posts }))
                .catch((err) => ({ ...state, error: err.message }));
        case "fetch-user":
            return fetch(`/api/users/${action.userId}`)
                .then((r) => r.json())
                .then((user) => ({ ...state, users: [user] }))
                .catch((err) => ({ ...state, error: err.message }));
    }
}

// Benefit: Actions are serializable (JSON), loggable, replayable, undoable
// This is exactly how Redux works -- actions are defunctionalized callbacks
```

## The Key Trade-off

| Property | Refunctionalized (higher-order) | Defunctionalized (data type) |
|----------|-------------------------------|------------------------------|
| Adding variants | **Open** -- just pass a new lambda | **Closed** -- must modify data type + apply |
| Serialization | Hard (can't serialize closures safely) | **Trivial** (it's just data) |
| Comparison | Hard (closure equality is undecidable) | **Trivial** (structural equality) |
| Inlining | Easy (compiler does it) | Requires manual apply expansion |
| Remote execution | Impossible (can't send lambdas over wire) | **Possible** (send the data) |

## Co-Change Signals

| Signal | What's happening | Direction |
|--------|-----------------|-----------|
| A higher-order function has a fixed, small set of callbacks passed to it across the codebase | The callback set is closed; defunctionalize to make it serializable and auditable | **Defunctionalize** (higher-order -> data) |
| A data type + interpreter keeps gaining new variants | The variant set is open; refunctionalize to avoid modifying the data type | **Refunctionalize** (data -> higher-order) |
| A recursive function has an explicit stack that mirrors the call tree | The stack is a defunctionalized continuation; consider refunctionalizing for clarity | Either direction |
| A state machine has states that correspond to control-flow positions in a recursive function | The states are defunctionalized continuations | **Refunctionalize** to recover the recursive structure |
| A callback-based API requires defining structs/enums for every async operation | The callbacks are defunctionalized continuations; consider whether the data/interpreters are worth the boilerplate | **Defunctionalize** if serialization matters, else stay higher-order |

## Concrete Applications

| Application | How defunctionalization applies |
|-------------|--------------------------------|
| **Recursion to iteration** | CPS-convert the recursive function, then defunctionalize the continuation -> yields an explicit stack data type -> tail-recursive form -> loop |
| **Splitting computation across machines** | CPS-convert the local code, defunctionalize the continuation -> yields a serializable "what to do next" message that can be sent over the wire |
| **Non-blocking I/O** | Defunctionalize the continuation of a blocking read -> yields a state machine that receives notification when data arrives |
| **Web actions (continuation-based frameworks)** | Defunctionalize the continuation saved between HTTP requests -> yields a form field or cookie that encodes the next action |
| **Plugins / strategy pattern** | Defunctionalize the plugin interface -> yields a data type of all known strategies + an interpreter |
| **Type-level programming** (higher-kinded polymorphism) | Defunctionalize at the type level to encode higher-kinded types in languages without native support (Lightweight Higher-Kinded Polymorphism) |
| **Visitor pattern** | The Visitor is a defunctionalization of pattern matching / algebraic type recursion |

## Relationship to Other Catalogs

Koppel argues that most refactorings in Fowler's catalog and Refactoring.Guru
are surface-level instances of deeper algebraic laws. Defunctionalization is one
such law -- it explains the structural relationship between:
- A lambda and an enum variant
- A callback and a message
- A continuation and a stack frame
- A strategy and a strategy selector
- A visitor and a recursive type traversal

Learning to recognize these isomorphic pairs eliminates the need to memorize
catalog entries. When you encounter a design choice between "function-passing"
and "data-dispatching," you are looking at a defunctionalization boundary.

## See Also

- [refactoring-catalog.md](refactoring-catalog.md) -- main refactoring catalog with paradigm-specific maps
- [reflect.md](reflect.md) -- the Reflect phase that triggers refactoring decisions
