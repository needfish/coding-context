# ------------------------------------------------------------------------------
#  refactoring-catalog.md
#  Reference to refactoring catalogs across paradigms  --  solution
#  transformations for problems found during Reflect Phase git history
#  analysis.
# ------------------------------------------------------------------------------

# Refactoring Catalog Reference

Refactoring operates at two levels:

| Level | Scope | Reference |
|-------|-------|-----------|
| **File / module** | Moving code between files, deciding when to extract or merge modules | Evan Czaplicki, "The Life of a File" -- when to split, when to keep together |
| **Within-file** | Code transformations inside a module (extract method, inline class, etc.) | Catalogs below (Refactoring.Guru, Kent, Data-Oriented) |

## File / Module Level: Evan Czaplicki's Split Strategy

Evan Czaplicki (creator of Elm) describes a strategy for file evolution in
"The Life of a File" (Elm Conf 2017, <https://www.youtube.com/watch?v=XpDsk374LDE>):

1. **Start in one file.** Do not pre-split. Let the code grow until a coherent
   abstraction emerges.
2. **Extract when the abstraction is clear.** Pull the abstraction into its own
   file when its interface is stable and it has a single responsibility.
3. **Split when a module changes for multiple reasons.** If git co-change
   analysis shows files in the same module never co-change, the module has
   multiple responsibilities -- split it.
4. **Merge when a cluster spans modules.** If git co-change analysis shows
   files in different modules always change together, the boundary is wrong --
   merge them into one module.

The trigger is always **observed co-change**, not anticipated future need.
Do not split or merge based on speculation.

**Connection to data representation:** The same exploration applies to data models.
Czaplicki's talk walks through concrete Elm examples of checkbox lists that look
identical but demand different data representations:

- **User settings** (notifications, autoplay, location) -- simple boolean toggles,
  modeled as a record of Bool fields
- **Fruit selection with a limit** (max 2 items) -- needs to enforce count constraints
  at the type level, modeled as a custom type or a set with a size invariant

The talk demonstrates multiple Elm type-level representations for each -- records,
dictionaries, custom types with different shapes -- as a concrete catalog of design
alternatives for the same UI pattern. This is the practical counterpart of William
Kent's systematic catalog of 37+ design options for a single fact (see
[many-forms.md](many-forms.md)).

Key insight from Czaplicki: **not everything that looks the same is the same thing.**
Two checkbox lists may appear identical, but different domain constraints demand
different data models. Exploring all the possible representations before committing
avoids premature abstraction that serves no concrete use case.

When a module's internal data model becomes awkward during git co-change analysis,
revisit the representation -- Kent's catalog shows the alternatives, Czaplicki's
examples demonstrate them in code.

Three catalogs for within-file transformations, organized by paradigm:

| Catalog | Paradigm | Scope |
|---------|----------|-------|
| [Refactoring.Guru](https://refactoring.guru/refactoring/catalog) | Object-oriented | Classes, objects, inheritance, polymorphism |
| [Functional Refactorings (Kent)](https://www.cs.kent.ac.uk/projects/refactor-fp/publications/refactoring-draft-catalogue.pdf) | Functional / algebraic | Data types, higher-order functions, monads, recursion |
| [Data-Oriented Design](https://www.dataorienteddesign.com/dodmain/node16.html) | Data-oriented | Transforms, tables, parallel processing |

---

## Cross-Paradigm: Defunctionalization / Refunctionalization

**Author:** Jimmy Koppel (MIT, pathsensitive.com)
**Sources:**
- Talk: "The Best Refactoring You've Never Heard Of" -- <https://www.youtube.com/watch?v=vNwukfhsOME>
- Blog post: <https://www.pathsensitive.com/2019/07/the-best-refactoring-youve-never-heard.html>
- SIGPLAN article: <https://blog.sigplan.org/2019/12/30/defunctionalization-everybody-does-it-nobody-talks-about-it/>

This is not a catalog entry but a **primordial transformation** that explains many
seemingly-unrelated refactorings across all paradigms. Understanding
defunctionalization lets you derive design alternatives mechanically rather than
relying on memorized catalog entries.

### What It Is

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

### The Key Trade-off

| Property | Refunctionalized (higher-order) | Defunctionalized (data type) |
|----------|-------------------------------|------------------------------|
| Adding variants | **Open** -- just pass a new lambda | **Closed** -- must modify data type + apply |
| Serialization | Hard (can't serialize closures safely) | **Trivial** (it's just data) |
| Comparison | Hard (closure equality is undecidable) | **Trivial** (structural equality) |
| Inlining | Easy (compiler does it) | Requires manual apply expansion |
| Remote execution | Impossible (can't send lambdas over wire) | **Possible** (send the data) |

### Co-Change Signals

| Signal | What's happening | Direction |
|--------|-----------------|-----------|
| A higher-order function has a fixed, small set of callbacks passed to it across the codebase | The callback set is closed; defunctionalize to make it serializable and auditable | **Defunctionalize** (higher-order -> data) |
| A data type + interpreter keeps gaining new variants | The variant set is open; refunctionalize to avoid modifying the data type | **Refunctionalize** (data -> higher-order) |
| A recursive function has an explicit stack that mirrors the call tree | The stack is a defunctionalized continuation; consider refunctionalizing for clarity | Either direction |
| A state machine has states that correspond to control-flow positions in a recursive function | The states are defunctionalized continuations | **Refunctionalize** to recover the recursive structure |
| A callback-based API requires defining structs/enums for every async operation | The callbacks are defunctionalized continuations; consider whether the data/interpreters are worth the boilerplate | **Defunctionalize** if serialization matters, else stay higher-order |

### Concrete Applications

| Application | How defunctionalization applies |
|-------------|--------------------------------|
| **Recursion to iteration** | CPS-convert the recursive function, then defunctionalize the continuation -> yields an explicit stack data type -> tail-recursive form -> loop |
| **Splitting computation across machines** | CPS-convert the local code, defunctionalize the continuation -> yields a serializable "what to do next" message that can be sent over the wire |
| **Non-blocking I/O** | Defunctionalize the continuation of a blocking read -> yields a state machine that receives notification when data arrives |
| **Web actions (continuation-based frameworks)** | Defunctionalize the continuation saved between HTTP requests -> yields a form field or cookie that encodes the next action |
| **Plugins / strategy pattern** | Defunctionalize the plugin interface -> yields a data type of all known strategies + an interpreter |
| **Type-level programming** (higher-kinded polymorphism) | Defunctionalize at the type level to encode higher-kinded types in languages without native support (Lightweight Higher-Kinded Polymorphism) |
| **Visitor pattern** | The Visitor is a defunctionalization of pattern matching / algebraic type recursion |

### Relationship to Other Catalogs

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

---

---

## How to Use This File

During the **Reflect** phase, after running git co-change analysis:

1. Identify which paradigm your module uses (OOP, functional, or data-oriented)
2. Find your co-change signal in the corresponding map below
3. Open the linked refactoring page
4. Follow the Problem / Solution / Mechanics to apply the transformation
5. Re-run the co-change analysis to confirm the cluster is now contained

---

## Object-Oriented Co-Change Cluster Map (Refactoring.Guru)

Full catalog: <https://refactoring.guru/refactoring/catalog>

| Co-change signal | Structural cause | Refactoring | Link |
|---|---|---|---|
| Files in different modules always change together | A method belongs in the consumer's module, not the producer's | **Move Method** | <https://refactoring.guru/move-method> |
| A field is updated alongside methods in another module | The field's lifecycle is tied to a different module | **Move Field** | <https://refactoring.guru/move-field> |
| A group of fields and methods changes together across modules | An implicit class is split across module boundaries | **Extract Class** | <https://refactoring.guru/extract-class> |
| A module has only one or two methods used by callers, rest is unused | The module no longer justifies its own boundary | **Inline Class** | <https://refactoring.guru/inline-class> |
| Module A calls into module B only to reach module C | The middleman adds no value | **Remove Middle Man** | <https://refactoring.guru/remove-middle-man> |
| Two modules share a common subset of fields and behavior | A shared base is needed | **Extract Superclass** | <https://refactoring.guru/extract-superclass> |
| Multiple clients depend on a subset of a module's interface | The shared portion should be its own contract | **Extract Interface** | <https://refactoring.guru/extract-interface> |
| A cluster spans modules but none owns the coupling | An intermediary or event channel is needed | **Introduce Local Extension** | <https://refactoring.guru/introduce-local-extension> |
| A module changes for multiple unrelated reasons | Two responsibilities are fused in one boundary | **Extract Class** (first split), then **Move Method** / **Move Field** per piece | <https://refactoring.guru/extract-class> |
| A utility module accumulates methods from every cluster | The utility is a dumping ground | **Extract Class** to group related utilities, or **Inline Class** if single-use | <https://refactoring.guru/extract-class> |
| A small stable cluster never co-changes with its parent module | The cluster should be promoted to its own module | **Extract Class** -> **Move Method** / **Move Field** to populate it | <https://refactoring.guru/extract-class> |
| A bidirectional co-change dependency between two modules | Unnecessary bidirectional association | **Change Bidirectional Association to Unidirectional** | <https://refactoring.guru/change-bidirectional-association-to-unidirectional> |
| Files change together but have no import/export relationship | Hidden implicit coupling via shared global state or side effects | First **Encapsulate Field**, then **Move Method** to bring the coupled logic into one module | <https://refactoring.guru/encapsulate-field> |

### Full Refactoring.Guru Technique Groups

#### Composing Methods
<https://refactoring.guru/refactoring/techniques/composing-methods>

- **Extract Method** -- turn a code fragment into its own method
- **Inline Method** -- replace a method call with its body
- **Extract Variable** -- give a subexpression a name
- **Inline Temp** -- replace a variable with its expression
- **Replace Temp with Query** -- replace a temporary variable with a method
- **Split Temporary Variable** -- use a separate variable for each responsibility
- **Remove Assignments to Parameters** -- use a local variable instead
- **Introduce Parameter Object** -- bundle related parameters into an object
- **Substitute Algorithm** -- replace an algorithm with a clearer one

#### Moving Features between Objects
<https://refactoring.guru/refactoring/techniques/moving-features-between-objects>

- **Move Method** -- move a method to the class that uses it most
- **Move Field** -- move a field to the class that uses it most
- **Extract Class** -- split a class into two separate classes
- **Inline Class** -- merge a class into another
- **Hide Delegate** -- hide delegation behind a method
- **Remove Middle Man** -- remove a delegating method
- **Introduce Foreign Method** -- add a method to a utility class
- **Introduce Local Extension** -- extend a utility class

#### Organizing Data
<https://refactoring.guru/refactoring/techniques/organizing-data>

- **Self Encapsulate Field** -- access a field through getters/setters
- **Replace Data Value with Object** -- replace a data field with a whole object
- **Change Value to Reference** -- share identical objects as a single reference
- **Change Reference to Value** -- turn a reference into a value object
- **Replace Array with Object** -- replace an array with an object that has named fields
- **Duplicate Observed Data** -- separate domain data from GUI data
- **Change Unidirectional Association to Bidirectional** -- add a back-reference
- **Change Bidirectional Association to Unidirectional** -- remove an unused direction
- **Replace Magic Number with Symbolic Constant** -- give a number a name
- **Encapsulate Field** -- make a field private and provide accessors
- **Encapsulate Collection** -- make a collection's getter return read-only
- **Replace Type Code with Class** -- replace a type code field with a new class
- **Replace Type Code with Subclasses** -- replace type code with inheritance
- **Replace Type Code with State/Strategy** -- replace type code with a state object
- **Replace Subclass with Fields** -- replace constant-returning subclasses with fields in the parent

#### Simplifying Conditional Expressions
<https://refactoring.guru/refactoring/techniques/simplifying-conditional-expressions>

- **Decompose Conditional** -- extract complex conditionals into methods
- **Consolidate Conditional Expression** -- combine related conditions
- **Consolidate Duplicate Conditional Fragments** -- deduplicate repeated code in conditional branches
- **Remove Control Flag** -- use break/return instead of a flag variable
- **Replace Nested Conditional with Guard Clauses** -- return early for edge cases
- **Replace Conditional with Polymorphism** -- use subclass methods instead of conditionals
- **Introduce Null Object** -- replace null checks with a null object
- **Introduce Assertion** -- document an assumption with an assertion

#### Simplifying Method Calls
<https://refactoring.guru/refactoring/techniques/simplifying-method-calls>

- **Rename Method** -- give a method a name that reveals its purpose
- **Add Parameter** -- add a parameter to a method
- **Remove Parameter** -- remove a parameter from a method
- **Separate Query from Modifier** -- separate side-effecting code from querying code
- **Parameterize Method** -- unify several similar methods with a parameter
- **Replace Parameter with Explicit Methods** -- replace a parameter with separate methods
- **Preserve Whole Object** -- pass an entire object instead of individual fields
- **Replace Parameter with Method Call** -- replace a parameter by calling a method
- **Introduce Parameter Object** -- bundle many parameters into an object
- **Remove Setting Method** -- remove a setter that is never used outside the constructor
- **Hide Method** -- make a method private
- **Replace Constructor with Factory Method** -- use a factory method instead of `new`
- **Replace Error Code with Exception** -- use exceptions instead of error codes
- **Replace Exception with Test** -- use a conditional check instead of an exception

#### Dealing with Generalization
<https://refactoring.guru/refactoring/techniques/dealing-with-generalization>

- **Pull Up Field** -- move a field to a superclass
- **Pull Up Method** -- move a method to a superclass
- **Pull Up Constructor Body** -- move constructor code to a superclass
- **Push Down Method** -- move a method from a superclass to a subclass
- **Push Down Field** -- move a field from a superclass to a subclass
- **Extract Subclass** -- create a subclass for features used only in certain cases
- **Extract Superclass** -- create a shared superclass
- **Extract Interface** -- extract a shared interface
- **Collapse Hierarchy** -- merge a subclass with its superclass
- **Form Template Method** -- unify algorithm steps in a superclass
- **Replace Inheritance with Delegation** -- replace a subclass with a delegate field
- **Replace Delegation with Inheritance** -- turn a delegate into a subclass

---

## Functional Co-Change Cluster Map (Kent Catalog)

Full catalog PDF: <https://www.cs.kent.ac.uk/projects/refactor-fp/publications/refactoring-draft-catalogue.pdf>

These refactorings apply to functional / algebraic-style code (Haskell, OCaml, Elm,
PureScript, F#, Rust enum-heavy code, TypeScript discriminated unions).

| Co-change signal | Structural cause | Refactoring | Reference |
|---|---|---|---|
| Pattern match arms in different modules change together | A variant has been added to a data type but handling is scattered | **Algebraic to Abstract Type** (Ref 9) | Hide the data constructor, expose selectors/discriminators |
| A function and its helper always change together | The helper is only used by one function | **Lift / Demote** (Ref 2) | Move helper into the calling function's scope or promote it to top-level |
| A type synonym appears at every function boundary in a co-change cluster | The synonym hides a shared invariant | **Opaque Type Naming** (Ref 4) | Convert `type` to `data` / `newtype` so the type system enforces the boundary |
| Functions change when a data representation changes | The data type is exposed across module boundaries | **Algebraic to Abstract Type** (Ref 9) | Hide constructors, export operations only |
| A general function and its specialized callers always change together | The general function has only one concrete call site | **Generalise / Inline** (Ref 6) | Inline the general function or promote the specialized one |
| Two modules always change together when adding a new variant | The variant type is shared; adding a variant requires updating all consumers | **Algebraic to Existential Type** (Ref 11) | Replace the algebraic type with a typeclass/interface and distributed instances |
| A data type and all functions pattern-matching on it change together | The type is in one module, the functions in another | **Migrate Functionality** (Ref 10) | Move the functions into the type's module or vice versa |
| A function's return type handling changes across many callers | The return type is too specific or too general | **Enlarge Return Type** (Ref 15) | Change `T` to `Maybe T`, `[T]`, or `Either T E` |
| A monadic computation and its consumers change together | The monad is exposed across module boundaries | **Monadic Presentation** (Ref 18) | Wrap the computation in a monad, push the monad choice to the consumer |
| Recursive functions and their callers change together | The recursion pattern is replicated | **Recursion to Fold** (Ref 20) | Replace explicit recursion with `fold` / `reduce` / catamorphism |
| Multiple case-switch flags appear together across commits | Flag-based dispatch has unstable boundaries | **Case Switch to Layered** (Ref 21, 22) | Reorganize into layered case switches or separate functions per branch |

### Full Functional Refactoring Catalog (University of Kent, v1)

| # | Refactoring | Description |
|---|---|---|
| 1 | **Rename** | Change a function name to reflect its purpose |
| 2 | **Lift / Demote** | Move a definition between top-level and local scope |
| 3 | **Name a Type** | Introduce a type synonym for a domain concept |
| 4 | **Opaque Type Naming** | Convert `type` synonym to `data` / `newtype` for type safety |
| 5 | **Explicit or Implicit Arguments** | Convert between point-free and point-ful style (eta conversion) |
| 6 | **Generalise / Inline** | Extract a general function or inline a single-use abstraction |
| 7 | **Add / Remove Field Names** | Toggle between positional and named fields in data constructors |
| 8 | **Pattern Matching or Selectors** | Replace pattern matching with selector functions and discriminators |
| 9 | **Algebraic or Abstract Type** | Expose or hide a data type's constructor structure |
| 10 | **Migrate Functionality** | Move a function from client to the type's module |
| 11 | **Algebraic or Existential Type** | Switch between sum types and typeclass-based dispatch |
| 12 | **Replace Function by Constructor** | Promote a convenience function to a data constructor |
| 13 | **Layered Data Types** | Flatten or nest data type constructors |
| 14 | **Set Comprehensions** | Switch between list/set/monad comprehension syntax |
| 15 | **Enlarge Return Type** | Change return type to `Maybe`, `List`, or `Either` |
| 16 | **Reorganise Arguments** | Reorder, curry, or uncurry function arguments |
| 17 | **Add / Remove Arguments** | Add or remove function parameters |
| 18 | **Monadic Presentation** | Lift a pure function into a monadic context |
| 19 | **Iteration / Fixedpoint** | Switch between bounded iteration and fixed-point detection |
| 20 | **Recursion / Fold Operator** | Replace explicit recursion with a fold / catamorphism |
| 21 | **Case Switch Flags** | Consolidate flag-based dispatch into conditional functions |
| 22 | **Layered Case Switches** | Separate pattern match cases into conceptual layers |

---

## Data-Oriented Co-Change Cluster Map

Reference: <https://www.dataorienteddesign.com/dodmain/node16.html>

These patterns apply to data-oriented / ECS-style code where logic is organized
as transforms over tables of data rather than objects with methods.

| Co-change signal | Structural cause | Refactoring | Reference |
|---|---|---|---|
| A transform function and its output table change together | The transform is coupled to the table's schema | **A to B Transform** | Ensure transform is stateless; input table read-only, output table write-only |
| An in-place update and its callers change together | Side-effect ordering is implicit | **In Place Transform** -> **A to B** | Convert to double-buffered A to B transform for concurrent safety |
| A table is regenerated from the same source across multiple callers | The transform is being repeated | **Generative Transform** | Extract the generative function so it runs once, feeds all consumers |
| A table is queried in multiple orders across different commits | The sort requirement changed | **Sorted Table** or **Multi Sorted Table** | Add an index per query pattern instead of re-sorting each time |
| Parallel tasks and their result collection change together | The gather step is tightly coupled to task output format | **Gatherer** | Add a gatherer transform that collects parallel outputs into one table |
| A task dispatcher and its workers change together | Dispatch logic is hard-coded per task type | **Dispatcher** | Pre-process data into job tables; one table per transform type |
| A table and its aggregate/reduce operations always change together | The reduction logic is scattered across callers | **New Transform** | Extract the aggregate as its own transform operating on the same table |
| A data layout change propagates to every transform | The transform signature exposes individual fields instead of rows | **Schema Iterator** | Use iterators that traverse table rows, making transforms schema-independent |

### Full Data-Oriented Design Patterns

Reference: <https://www.dataorienteddesign.com/dodmain/node16.html>

| Pattern | Description |
|---|---|
| **A to B Transform** | Stateless function: reads from table A, writes to table B. No side effects. Basis for double-buffered state and parallel processing. |
| **In Place Transform** | Stateless function that modifies a table in place. Faster than A to B but cannot parallelize. |
| **Generative Transform** | Generates data from constants/algorithms instead of an input table. Used for procedural generation, file loading, bootstrapping. |
| **Sorted Table** | Container that keeps itself sorted. Prefer b-trees over binary searches for cache-friendliness. |
| **Multi Sorted Table** | Multiple sorted views of the same data via indirection indexes. Like database indexes per column. |
| **Gatherer** | Collects outputs from parallel tasks into one table. Can run concurrently with transforms. |
| **Tasker** | Splits a large table into pieces and runs transforms as concurrent tasks. |
| **Dispatcher** | Pre-processes a table into job tables (one per transform type) to avoid instruction cache misses from data-dependent dispatch. |

---

## Language-Specific Refactoring References

### Rust

- **Rust Book -- Refactoring to Improve Modularity**: <https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html>
  Splitting `main.rs` into `lib.rs`, separating concerns, extracting modules.
- **Rust Module Organization**: <https://mmapped.blog/posts/03-rust-packages-crates-modules>
  When to split packages vs. modules vs. crates. Dependency cycle avoidance.
- **Overcoming Cyclic Dependencies**: <https://www.slingacademy.com/article/overcoming-cyclic-dependencies-by-refactoring-rust-modules/>
  Practical strategies for breaking cycles via trait extraction, inversion, and re-organization.

### Python

- **Basedpyright strict mode** enforces many refactorings at compile time (no `Any`, strict type checks).
- Python's module system naturally supports **Extract Module** by moving functions into a new file.

### General

- **Martin Fowler's Catalog** (<https://refactoring.com/catalog/>) -- the original catalog, paradigm-neutral for most composing-method refactorings (Extract Function, Inline Variable, etc.).
- **Safety Enforcement Tool References** ([safety-tools.md](safety-tools.md)) -- linting, static analysis, and enforcement tools for JavaScript, C/C++, Rust, and Python.
