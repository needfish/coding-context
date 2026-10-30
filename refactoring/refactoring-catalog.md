# Refactoring Catalog Reference

Refactoring operates at two levels:

| Level | Scope | Reference |
|---|---|---|
| File / module | Moving code between files, deciding when to extract or merge modules | Evan Czaplicki, "The Life of a File" |
| Within-file | Code transformations inside a module | Catalogs below (Refactoring.Guru, Kent, Data-Oriented) |

## File / Module Level: Evan Czaplicki's Split Strategy

Source: "The Life of a File" (Elm Conf 2017, https://www.youtube.com/watch?v=XpDsk374LDE)

1. Start in one file. Do not pre-split.
2. Extract when the abstraction is clear.
3. Split when a module changes for multiple reasons (git co-change analysis shows files in the same module never co-change).
4. Merge when a cluster spans modules (files in different modules always change together).

The trigger is always observed co-change, not anticipated future need.

Connection to data representation: Czaplicki's talk walks through Elm examples of checkbox lists that look identical but demand different data representations:
- User settings (notifications, autoplay, location) -- simple boolean toggles, modeled as a record of Bool fields
- Fruit selection with a limit (max 2 items) -- needs to enforce count constraints at the type level

Key insight: not everything that looks the same is the same thing.

When a module's internal data model becomes awkward during git co-change analysis, revisit the representation. Kent's catalog ([many-forms.md](../modeling/many-forms.md)) shows the alternatives; Czaplicki's examples demonstrate them in code.

Three catalogs for within-file transformations, organized by paradigm:

| Catalog | Paradigm | Scope |
|---|---|---|
| [Refactoring.Guru](https://refactoring.guru/refactoring/catalog) | Object-oriented | Classes, objects, inheritance, polymorphism |
| [Functional Refactorings (Kent)](https://www.cs.kent.ac.uk/projects/refactor-fp/publications/refactoring-draft-catalogue.pdf) | Functional / algebraic | Data types, higher-order functions, monads, recursion |
| [Data-Oriented Design](https://www.dataorienteddesign.com/dodmain/node16.html) | Data-oriented | Transforms, tables, parallel processing |

## Cross-Paradigm: Defunctionalization / Refunctionalization

See [defunctionalization.md](defunctionalization.md) for trade-off analysis, co-change signals, code examples, and concrete applications.

## How to Use This File

During the Reflect phase, after running git co-change analysis:
1. Identify which paradigm the module uses (OOP, functional, or data-oriented)
2. Find the co-change signal in the corresponding map below
3. Open the linked refactoring page
4. Follow the Problem / Solution / Mechanics to apply the transformation
5. Re-run the co-change analysis to confirm the cluster is now contained

## Object-Oriented Co-Change Cluster Map

Full catalog: https://refactoring.guru/refactoring/catalog

| Co-change signal | Structural cause | Refactoring | Link |
|---|---|---|---|
| Files in different modules always change together | A method belongs in the consumer's module, not the producer's | Move Method | https://refactoring.guru/move-method |
| A field is updated alongside methods in another module | The field's lifecycle is tied to a different module | Move Field | https://refactoring.guru/move-field |
| A group of fields and methods changes together across modules | An implicit class is split across module boundaries | Extract Class | https://refactoring.guru/extract-class |
| A module has only one or two methods used by callers, rest is unused | The module no longer justifies its own boundary | Inline Class | https://refactoring.guru/inline-class |
| Module A calls into module B only to reach module C | The middleman adds no value | Remove Middle Man | https://refactoring.guru/remove-middle-man |
| Two modules share a common subset of fields and behavior | A shared base is needed | Extract Superclass | https://refactoring.guru/extract-superclass |
| Multiple clients depend on a subset of a module's interface | The shared portion should be its own contract | Extract Interface | https://refactoring.guru/extract-interface |
| A cluster spans modules but none owns the coupling | An intermediary or event channel is needed | Introduce Local Extension | https://refactoring.guru/introduce-local-extension |
| A module changes for multiple unrelated reasons | Two responsibilities are fused in one boundary | Extract Class (first split), then Move Method / Move Field per piece | https://refactoring.guru/extract-class |
| A utility module accumulates methods from every cluster | The utility is a dumping ground | Extract Class to group related utilities, or Inline Class if single-use | https://refactoring.guru/extract-class |
| A small stable cluster never co-changes with its parent module | The cluster should be promoted to its own module | Extract Class -> Move Method / Move Field to populate it | https://refactoring.guru/extract-class |
| A bidirectional co-change dependency between two modules | Unnecessary bidirectional association | Change Bidirectional Association to Unidirectional | https://refactoring.guru/change-bidirectional-association-to-unidirectional |
| Files change together but have no import/export relationship | Hidden implicit coupling via shared global state or side effects | Encapsulate Field, then Move Method | https://refactoring.guru/encapsulate-field |

### Refactoring.Guru Technique Groups

#### Composing Methods
https://refactoring.guru/refactoring/techniques/composing-methods

- Extract Method, Inline Method, Extract Variable, Inline Temp, Replace Temp with Query, Split Temporary Variable, Remove Assignments to Parameters, Introduce Parameter Object, Substitute Algorithm

#### Moving Features between Objects
https://refactoring.guru/refactoring/techniques/moving-features-between-objects

- Move Method, Move Field, Extract Class, Inline Class, Hide Delegate, Remove Middle Man, Introduce Foreign Method, Introduce Local Extension

#### Organizing Data
https://refactoring.guru/refactoring/techniques/organizing-data

- Self Encapsulate Field, Replace Data Value with Object, Change Value to Reference, Change Reference to Value, Replace Array with Object, Duplicate Observed Data, Change Unidirectional Association to Bidirectional, Change Bidirectional Association to Unidirectional, Replace Magic Number with Symbolic Constant, Encapsulate Field, Encapsulate Collection, Replace Type Code with Class, Replace Type Code with Subclasses, Replace Type Code with State/Strategy, Replace Subclass with Fields

#### Simplifying Conditional Expressions
https://refactoring.guru/refactoring/techniques/simplifying-conditional-expressions

- Decompose Conditional, Consolidate Conditional Expression, Consolidate Duplicate Conditional Fragments, Remove Control Flag, Replace Nested Conditional with Guard Clauses, Replace Conditional with Polymorphism, Introduce Null Object, Introduce Assertion

#### Simplifying Method Calls
https://refactoring.guru/refactoring/techniques/simplifying-method-calls

- Rename Method, Add Parameter, Remove Parameter, Separate Query from Modifier, Parameterize Method, Replace Parameter with Explicit Methods, Preserve Whole Object, Replace Parameter with Method Call, Introduce Parameter Object, Remove Setting Method, Hide Method, Replace Constructor with Factory Method, Replace Error Code with Exception, Replace Exception with Test

#### Dealing with Generalization
https://refactoring.guru/refactoring/techniques/dealing-with-generalization

- Pull Up Field, Pull Up Method, Pull Up Constructor Body, Push Down Method, Push Down Field, Extract Subclass, Extract Superclass, Extract Interface, Collapse Hierarchy, Form Template Method, Replace Inheritance with Delegation, Replace Delegation with Inheritance

## Functional Co-Change Cluster Map

Full catalog PDF: https://www.cs.kent.ac.uk/projects/refactor-fp/publications/refactoring-draft-catalogue.pdf

| Co-change signal | Structural cause | Refactoring | Reference |
|---|---|---|---|
| Pattern match arms in different modules change together | A variant has been added to a data type but handling is scattered | Algebraic to Abstract Type (Ref 9) | Hide the data constructor, expose selectors/discriminators |
| A function and its helper always change together | The helper is only used by one function | Lift / Demote (Ref 2) | Move helper into the calling function's scope or promote it to top-level |
| A type synonym appears at every function boundary in a co-change cluster | The synonym hides a shared invariant | Opaque Type Naming (Ref 4) | Convert type to data / newtype so the type system enforces the boundary |
| Functions change when a data representation changes | The data type is exposed across module boundaries | Algebraic to Abstract Type (Ref 9) | Hide constructors, export operations only |
| A general function and its specialized callers always change together | The general function has only one concrete call site | Generalise / Inline (Ref 6) | Inline the general function or promote the specialized one |
| Two modules always change together when adding a new variant | The variant type is shared; adding a variant requires updating all consumers | Algebraic to Existential Type (Ref 11) | Replace the algebraic type with a typeclass/interface and distributed instances |
| A data type and all functions pattern-matching on it change together | The type is in one module, the functions in another | Migrate Functionality (Ref 10) | Move the functions into the type's module or vice versa |
| A function's return type handling changes across many callers | The return type is too specific or too general | Enlarge Return Type (Ref 15) | Change T to Maybe T, [T], or Either T E |
| A monadic computation and its consumers change together | The monad is exposed across module boundaries | Monadic Presentation (Ref 18) | Wrap the computation in a monad, push the monad choice to the consumer |
| Recursive functions and their callers change together | The recursion pattern is replicated | Recursion to Fold (Ref 20) | Replace explicit recursion with fold / reduce / catamorphism |
| Multiple case-switch flags appear together across commits | Flag-based dispatch has unstable boundaries | Case Switch to Layered (Ref 21, 22) | Reorganize into layered case switches or separate functions per branch |

### Full Functional Refactoring Catalog

| # | Refactoring | Description |
|---|---|---|
| 1 | Rename | Change a function name to reflect its purpose |
| 2 | Lift / Demote | Move a definition between top-level and local scope |
| 3 | Name a Type | Introduce a type synonym for a domain concept |
| 4 | Opaque Type Naming | Convert type synonym to data / newtype for type safety |
| 5 | Explicit or Implicit Arguments | Convert between point-free and point-ful style (eta conversion) |
| 6 | Generalise / Inline | Extract a general function or inline a single-use abstraction |
| 7 | Add / Remove Field Names | Toggle between positional and named fields in data constructors |
| 8 | Pattern Matching or Selectors | Replace pattern matching with selector functions and discriminators |
| 9 | Algebraic or Abstract Type | Expose or hide a data type's constructor structure |
| 10 | Migrate Functionality | Move a function from client to the type's module |
| 11 | Algebraic or Existential Type | Switch between sum types and typeclass-based dispatch |
| 12 | Replace Function by Constructor | Promote a convenience function to a data constructor |
| 13 | Layered Data Types | Flatten or nest data type constructors |
| 14 | Set Comprehensions | Switch between list/set/monad comprehension syntax |
| 15 | Enlarge Return Type | Change return type to Maybe, List, or Either |
| 16 | Reorganise Arguments | Reorder, curry, or uncurry function arguments |
| 17 | Add / Remove Arguments | Add or remove function parameters |
| 18 | Monadic Presentation | Lift a pure function into a monadic context |
| 19 | Iteration / Fixedpoint | Switch between bounded iteration and fixed-point detection |
| 20 | Recursion / Fold Operator | Replace explicit recursion with a fold / catamorphism |
| 21 | Case Switch Flags | Consolidate flag-based dispatch into conditional functions |
| 22 | Layered Case Switches | Separate pattern match cases into conceptual layers |

## Data-Oriented Co-Change Cluster Map

Reference: https://www.dataorienteddesign.com/dodmain/node16.html

| Co-change signal | Structural cause | Refactoring | Reference |
|---|---|---|---|
| A transform function and its output table change together | The transform is coupled to the table's schema | A to B Transform | Ensure transform is stateless; input table read-only, output table write-only |
| An in-place update and its callers change together | Side-effect ordering is implicit | In Place Transform -> A to B | Convert to double-buffered A to B transform for concurrent safety |
| A table is regenerated from the same source across multiple callers | The transform is being repeated | Generative Transform | Extract the generative function so it runs once, feeds all consumers |
| A table is queried in multiple orders across different commits | The sort requirement changed | Sorted Table or Multi Sorted Table | Add an index per query pattern instead of re-sorting each time |
| Parallel tasks and their result collection change together | The gather step is tightly coupled to task output format | Gatherer | Add a gatherer transform that collects parallel outputs into one table |
| A task dispatcher and its workers change together | Dispatch logic is hard-coded per task type | Dispatcher | Pre-process data into job tables; one table per transform type |
| A table and its aggregate/reduce operations always change together | The reduction logic is scattered across callers | New Transform | Extract the aggregate as its own transform operating on the same table |
| A data layout change propagates to every transform | The transform signature exposes individual fields instead of rows | Schema Iterator | Use iterators that traverse table rows, making transforms schema-independent |

### Data-Oriented Design Patterns

Reference: https://www.dataorienteddesign.com/dodmain/node16.html

| Pattern | Description |
|---|---|
| A to B Transform | Stateless function: reads from table A, writes to table B. No side effects. |
| In Place Transform | Stateless function that modifies a table in place. Faster than A to B but cannot parallelize. |
| Generative Transform | Generates data from constants/algorithms instead of an input table. |
| Sorted Table | Container that keeps itself sorted. |
| Multi Sorted Table | Multiple sorted views of the same data via indirection indexes. |
| Gatherer | Collects outputs from parallel tasks into one table. |
| Tasker | Splits a large table into pieces and runs transforms as concurrent tasks. |
| Dispatcher | Pre-processes a table into job tables (one per transform type) to avoid instruction cache misses. |

## Language-Specific Refactoring References

### Rust

- Rust Book -- Refactoring to Improve Modularity: https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html
- Rust Module Organization: https://mmapped.blog/posts/03-rust-packages-crates-modules
- Overcoming Cyclic Dependencies: https://www.slingacademy.com/article/overcoming-cyclic-dependencies-by-refactoring-rust-modules/

### Python

- Basedpyright strict mode enforces many refactorings at compile time.
- Python's module system naturally supports Extract Module.

### General

- Martin Fowler's Catalog: https://refactoring.com/catalog/
- Safety Enforcement Tool References: [../toolchain/safety-tools.md](../toolchain/safety-tools.md)
