# How to Use This Repository

Guide to using the software problem pattern repository. Open during the Model phase.

## Project Type Reference

| Project type | Core patterns |
|---|---|
| CRUD web app | pluto (parsing), saturn (routing), jupiter (validation), mars (persistence), neptune (transformation), uranus (authentication), ganymede (output formatting), mercury (display) |
| Compiler / interpreter | pluto (parsing), jupiter (validation), eris (pipeline), neptune (transformation), ganymede (output formatting), mercury (display) |
| Editor / note app | pluto (parsing), pandora (workpieces), mercury (display), mars (persistence), titan (highlighting), ganymede (output formatting) |
| CLI data tool | pluto (parsing), europa (config), venus (filtering), neptune (transformation), ganymede (output formatting), mercury (display) |
| Embedded sensor / IoT node | pluto (parsing), saturn (routing), neptune (transformation), mars (persistence), phoebe (resource lifecycle), pan (scheduling), ganymede (output formatting) |
| E-commerce / order processing | pluto (parsing), charon (workflow), mars (persistence), saturn (routing), jupiter (validation), pan (scheduling), ganymede (output formatting), mercury (display) |
| Chat / real-time messaging | pluto (parsing), hyperion (listener), saturn (routing), mars (persistence), pan (scheduling), ganymede (output formatting), mercury (display) |
| Data pipeline / ETL | pluto (parsing), eris (pipeline), neptune (transformation), mars (persistence), charon (workflow), ganymede (output formatting) |
| Game engine | pluto (parsing), saturn (routing), mercury (display), mars (persistence), neptune (transformation), pan (scheduling), ganymede (output formatting) |
| Testing framework | pluto (parsing), callisto (filesystem walk), saturn (routing), eris (pipeline), ganymede (output formatting), mercury (display), iota (command execution) |
| API gateway / reverse proxy | pluto (parsing), hyperion (listener), saturn (routing), uranus (authentication), neptune (transformation), pan (scheduling), ganymede (output formatting) |
| File sync / backup | pluto (parsing), callisto (filesystem walk), venus (filtering), mars (persistence), neptune (transformation), charon (workflow), ganymede (output formatting) |
| Search engine / indexer | pluto (parsing), callisto (filesystem walk), neptune (transformation), mercury (display), ganymede (output formatting), thebe (compaction) |

## Section Reference

| Phase | Sections to read | What to extract |
|---|---|---|
| Model | Problem, Input, Output, Observable Behavior | What it does, what it takes, what it produces |
| Build | Minimal Code, Start With | Minimal implementation, skeleton structure |
| Evaluate | Failure Modes, Signs This Is Present | Edge cases, comparison against real code |
| Reflect | Split Into Its Own Module When, Common Mistakes | Refactoring triggers, lessons learned |
| Reflect | [refactoring/refactoring-catalog.md](refactoring/refactoring-catalog.md) | Choose the right refactoring technique for a co-change cluster |

### Fast assessment order

```
Problem                    ->  Does this describe the situation?
Signs This Is Present      ->  Does the code smell like this?
Minimal Code               ->  What is the simplest implementation?
Split Into Its Own Module  ->  When to refactor?
Common Mistakes            ->  What mistakes do people make?
```

### Full understanding order

```
Problem                    ->  One-sentence definition
Input                      ->  What data does it take?
Output                     ->  What does it produce?
Observable Behavior        ->  What happens when it runs?
Minimal Code               ->  What is the simplest working version?
Failure Modes              ->  What goes wrong?
Design Trade-offs          ->  What choices affect behavior?
Signs This Is Present      ->  How to recognise it?
Common Mistakes            ->  What to avoid?
```

## Navigation

### Step 1: Read the core patterns for the project type

Open the patterns listed under "Core patterns" in the table above. During Model, focus on Problem, Input, Output, and Observable Behavior. During Build, follow Minimal Code.

### Step 2: Decompose into subproblems

As each pattern is read, the system may have multiple problem types. Each becomes a subproblem with its own module. Name the module after the planet. The planet name is a foreign key back to the pattern.

### Step 3: Watch for co-occurring patterns

Common Clusters in [pattern-table.md](pattern-table.md) shows patterns that travel together.

Example: saturn (routing) -> also look at hyperion (listener), jupiter (validation). When a pattern is split into its own module, check its neighbors.

### Step 4: Recognize splitting boundaries during Reflect

| Observation | Pattern says |
|---|---|
| An if-chain grows to 15 branches | saturn: split when 20+ routes |
| Save/load is a single JSON dump | mars: split when incremental saves needed |
| Filtering becomes user-configurable | venus: split when predicates are dynamic |
| Multiple stages with shared context | eris: split when stages need independent testing |
| An edit application is full-string replace | pandora: split when large edits are slow |
| Raw file or network data arrives in bytes | pluto: split when encoding detection or format sniffing is needed |

After refactoring, run a 5 whys analysis on the issue that triggered the split. If all root causes are resolved, the design is stable for this pattern. If not, loop back through Model with the new understanding.

### Step 5: Apply the right refactoring technique

After git co-change analysis identifies module-boundary violations, open [refactoring/refactoring-catalog.md](refactoring/refactoring-catalog.md) and find the row that matches the co-change signal.

| Paradigm | Catalog | Use when |
|---|---|---|
| Object-oriented | [Refactoring.Guru](https://refactoring.guru/refactoring/catalog) | Code uses classes, objects, inheritance, polymorphism |
| Functional / algebraic | [Kent Catalog](https://www.cs.kent.ac.uk/projects/refactor-fp/publications/refactoring-draft-catalogue.pdf) | Code uses sum types, pattern matching, higher-order functions, monads |
| Data-oriented | [Data-Oriented Design](https://www.dataorienteddesign.com/dodmain/node16.html) | Code organizes logic as transforms over tables of data |

## Example: CLI File Search Tool

### Phase 1: Model

Project type: CLI data tool -> europa, pluto, venus, ganymede, callisto, neptune.

| Planet | Module name | Problem | Key invariant from pattern |
|---|---|---|---|
| europa | config/ | Assemble CLI flags + env vars + file config | Higher-priority sources override lower on a per-field basis |
| pluto | input/ | Read and parse files or stdin | Encoding determined before character-level parsing |
| venus | filter/ | Match filenames against a pattern | Every result satisfies the predicate |
| ganymede | output/ | Print results to terminal | Format transformations lossless for the info the user needs |
| callisto | walk/ | Traverse directory tree | Visit every reachable file exactly once |
| neptune | transform/ | Convert file paths to display paths | Transformation has no side effects on source |

### Phase 2: Build

```python
import os, re
pattern = re.compile(sys.argv[1])
results = []
for root, dirs, files in os.walk("."):
    for f in files:
        if pattern.search(f):
            results.append(os.path.join(root, f))
for r in results:
    print(r)
```

### Phase 3: Evaluate

| Pattern | Failure mode | Tool |
|---|---|---|
| callisto | Symlink loop | Not yet handled -- add a visited-set guard |
| venus | Predicate side effects | Safe -- re.search is pure |
| ganymede | Format loss | Safe -- plain text is lossless |

Check [examined-projects.md](examined-projects.md) to see how `fd` handles similar issues: it uses `ignore::WalkParallel` for concurrent traversal with built-in symlink protection.

### Phase 4: Reflect

No triggers fired. The tool is 50 lines and meets all current needs.

### Next Iteration: Trigger Fires

Trigger: users want --exclude-dir node_modules.

Open callisto -> split when exclusions are needed.

```python
def walk(root, should_skip=None):
    for dirpath, dirnames, filenames in os.walk(root):
        dirnames[:] = [d for d in dirnames if not should_skip(d)]
        yield from (os.path.join(dirpath, f) for f in filenames)
```

### Subsequent Iterations

| Trigger | Pattern | Next increment |
|---|---|---|
| Users want --json / --csv | ganymede | display_json(), display_csv() separated from plain output |
| Users want *.md / *.py flag filters | venus | Parse filter expressions from CLI args |
| Users pipe through stdin in varied encodings | pluto | Add input source abstraction, encoding detection |
| Path format tokens keep changing | neptune | Extract path formatter, unit-test independently |

After each increment, run Evaluate (check failure modes) and Reflect (5 whys on any issues). If all root causes are resolved across all patterns, the design is stable. If not, loop back to Model.

## When to Ignore a Pattern

| Only at small scale | Only at large scale |
|---|---|
| venus (one-line filtering) | thebe (concurrent compaction) |
| ganymede (plain text output) | phoebe (resource lifecycle) |
| mars (JSON file save) | charon (durable workflow) |
| pluto (single read()) | (applies at all scales) |

## Key Mindset Shifts

| Old thinking | New thinking |
|---|---|
| "I need a design pattern." | "I have a problem -- which pattern describes it?" |
| "Let me pick the right abstraction upfront." | "Let me start minimal and observe when the naive form breaks." |
| "A router is an abstract dispatch mechanism." | "A router is a hash map from key to handler." |
| "I need undo/redo, so I need a transaction system." | "I need undo/redo, so I start with a single string and full-snapshot undo." |
| "I should build a pipeline framework for my compiler." | "I should call three functions in sequence and check errors between them." |
| "I need a streaming parser because the input is large." | "I should read the whole file first and measure before I optimise." |
