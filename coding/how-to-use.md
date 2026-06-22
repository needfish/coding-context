# ------------------------------------------------------------------------------
#  how-to-use.md
#  Guide to using the software problem pattern repository.
# ------------------------------------------------------------------------------

# How to Use This Repository

## Start with Your Project Type

Find your project type during the **Model** phase (Model  ->  Build  ->  Evaluate  ->  Reflect). The table shows which patterns your system will instantiate as you decompose it. Core patterns define the structure; supporting patterns emerge as the system grows.

| Project type                         | Core patterns                                                              |
|--------------------------------------|---------------------------------------------------------------------------|
| CRUD web app                         | pluto (parsing), saturn (routing), jupiter (validation), mars (persistence), neptune (transformation), uranus (authentication), ganymede (output formatting), mercury (display) |
| Compiler / interpreter               | pluto (parsing), jupiter (validation), eris (pipeline), neptune (transformation), ganymede (output formatting), mercury (display) |
| Editor / note app                    | pluto (parsing), pandora (workpieces), mercury (display), mars (persistence), titan (highlighting), ganymede (output formatting) |
| CLI data tool                        | pluto (parsing), europa (config), venus (filtering), neptune (transformation), ganymede (output formatting), mercury (display) |
| Embedded sensor / IoT node           | pluto (parsing), saturn (routing), neptune (transformation), mars (persistence), phoebe (resource lifecycle), pan (scheduling), ganymede (output formatting) |
| E-commerce / order processing        | pluto (parsing), charon (workflow), mars (persistence), saturn (routing), jupiter (validation), pan (scheduling), ganymede (output formatting), mercury (display) |
| Chat / real-time messaging           | pluto (parsing), hyperion (listener), saturn (routing), mars (persistence), pan (scheduling), ganymede (output formatting), mercury (display) |
| Data pipeline / ETL                  | pluto (parsing), eris (pipeline), neptune (transformation), mars (persistence), charon (workflow), ganymede (output formatting) |
| Game engine                          | pluto (parsing), saturn (routing), mercury (display), mars (persistence), neptune (transformation), pan (scheduling), ganymede (output formatting) |
| Testing framework                    | pluto (parsing), callisto (filesystem walk), saturn (routing), eris (pipeline), ganymede (output formatting), mercury (display), io (command execution) |
| API gateway / reverse proxy          | pluto (parsing), hyperion (listener), saturn (routing), uranus (authentication), neptune (transformation), pan (scheduling), ganymede (output formatting) |
| File sync / backup                   | pluto (parsing), callisto (filesystem walk), venus (filtering), mars (persistence), neptune (transformation), charon (workflow), ganymede (output formatting) |
| Search engine / indexer              | pluto (parsing), callisto (filesystem walk), neptune (transformation), mercury (display), ganymede (output formatting), thebe (compaction) |

Read the core patterns for your project type first. The sections below explain the pattern structure and how to use each section during the workflow.

---

## How to Read a Full Pattern File

Each pattern uses a consistent structure. Which sections you read depends on where you are in the workflow:

| Phase | Read these sections | What to extract |
|-------|--------------------|-----------------|
| **Model** | Domains, Invariants, Related Subproblems, Frame Concern | Types, state machines, subproblem decomposition |
| **Build** | Start with (minimal form), Signs you are in this pattern | Minimal implementation, skeleton structure |
| **Evaluate** | Known Failure Modes, Real project examples | Edge cases, comparison against real code |
| **Reflect** | Split into its own module when, Common traps | Refactoring triggers, lessons learned |
| **Reflect** | [refactoring-catalog.md](refactoring-catalog.md) | Choose the right refactoring technique for a co-change cluster |

**Order for first pass (fast assessment):**

```
When this pattern matters       ->   Does this describe my situation?
Signs you are in this pattern   ->   Does my code smell like this?
Start with (minimal form)       ->   What is the simplest implementation?
Split into its own module when  ->   When do I need to refactor?
Common traps                    ->   What mistakes do people make?
Real project examples           ->   Who else has solved this?
```

**Order for deeper understanding:**

```
[Title + Pressure Class]   ->   What kind of variation this pattern addresses
                             (Data / Behavior / Object / Rules)

Problem                    ->   One-sentence definition
Context                    ->   When does this arise?
Domains Involved           ->   What pieces participate? (causal / lexical / biddable)
Shared Phenomena           ->   What crosses the boundaries between pieces?
Invariants                 ->   What must always be true?
Known Failure Modes        ->   What goes wrong?
Forces                     ->   What tensions shape any solution?
Related Subproblems        ->   What other patterns co-occur?
------- PRACTICAL USE --------
Frame Concern              ->   The fundamental difficulty
```

---

## How to Navigate the Repository

**You do not need to read all 26 patterns.** The repository is designed for targeted lookup.

### Step 1: Read the core patterns for your project type

Open the patterns listed under "Core patterns" in the table above. During the **Model phase**, focus on Domains and Invariants to define your types and state machines. During **Build**, follow "Start with (minimal form)".

### Step 2: Decompose into subproblems

As you read each pattern, you will recognise that your system has multiple problem types. Each becomes a subproblem with its own module. Name the module after the planet  --  "this module routes messages" becomes `saturn/`, "this module persists state" becomes `mars/`. The planet name is a foreign key back to the pattern.

### Step 3: Watch for co-occurring patterns

The Common Clusters section in [pattern-table.md](pattern-table.md) shows patterns that travel together:

```
saturn (routing)        ->   also look at hyperion (listener), jupiter (validation)
pandora (workpieces)    ->   also look at mercury (display), mars (persistence)
callisto (walk)         ->   also look at venus (filtering), ganymede (output format)
hyperion (listener)     ->   also look at neptune (transformation), saturn (routing)
titan (highlighting)    ->   also look at rhea (detection)
eris (pipeline)         ->   also look at neptune (transformation) repeated N times
pluto (parsing)         ->   also look at jupiter (validation), neptune (transformation), mars (persistence)
janus (reduction)       ->   also look at venus (filtering), neptune (transformation), earth (sorting), eris (pipeline)
```

When you split a pattern into its own module, check its neighbors. If you are building a router, you almost certainly need validation and a listener too.

### Step 4: Recognize splitting boundaries during Reflect

The moment one of these triggers fires, open the corresponding pattern, read the "Split into its own module" section, and implement the next increment in the next Build iteration.

| You notice... | The pattern says... |
|--------------|-------------------|
| An if-chain grows to 15 branches | `saturn`: split when 20+ routes |
| Save/load is a single JSON dump | `mars`: split when incremental saves needed |
| Filtering becomes user-configurable | `venus`: split when predicates are dynamic |
| Multiple stages with shared context | `eris`: split when stages need independent testing |
| An edit application is full-string replace | `pandora`: split when large edits are slow |
| Raw file or network data arrives in bytes | `pluto`: split when encoding detection or format sniffing is needed |

After refactoring, run a 5 whys analysis on the issue that triggered the split. If all root causes are resolved, the design is stable for this pattern. If not, loop back through Model with the new understanding.

### Step 5: Apply the Right Refactoring Technique

After git co-change analysis identifies module-boundary violations, open
[refactoring-catalog.md](refactoring-catalog.md) and find the row that matches
your co-change signal. The catalog maps signals to techniques across three
paradigms:

| Paradigm | Catalog | Use when... |
|----------|---------|-------------|
| Object-oriented | [Refactoring.Guru](https://refactoring.guru/refactoring/catalog) | Your code uses classes, objects, inheritance, polymorphism |
| Functional / algebraic | [Kent Catalog](https://www.cs.kent.ac.uk/projects/refactor-fp/publications/refactoring-draft-catalogue.pdf) | Your code uses sum types, pattern matching, higher-order functions, monads |
| Data-oriented | [Data-Oriented Design](https://www.dataorienteddesign.com/dodmain/node16.html) | Your code organizes logic as transforms over tables of data |

Apply the refactoring, then re-run the co-change analysis to confirm the cluster
is now contained within a single module.

---

---

## Concrete Walkthrough: Building a CLI File Search Tool

### Phase 1: Model  --  Decompose

From the project type table:

> **CLI data tool**  ->  **europa**, **pluto**, **venus**, **ganymede**, **callisto**, **neptune**

You skim the six patterns, focusing on their Domains, Invariants, and Related Subproblems. You identify the subproblems and name your modules:

| Planet | Module name | Problem | Key invariant from pattern |
|--------|-------------|---------|---------------------------|
| **europa** | `config/` | Assemble CLI flags + env vars + file config | Higher-priority sources must override lower on a per-field basis |
| **pluto** | `input/` | Read and parse files or stdin | Encoding must be determined before character-level parsing begins |
| **venus** | `filter/` | Match filenames against a pattern | Every result must satisfy the predicate |
| **ganymede** | `output/` | Print results to terminal | Format transformations must be lossless for the info the user needs |
| **callisto** | `walk/` | Traverse directory tree | Must visit every reachable file exactly once |
| **neptune** | `transform/` | Convert file paths to display paths | Transformation must not have side effects on source |

You define the types and signatures for each module. The system now has a named decomposition.

### Phase 2: Build  --  Implement Minimal Forms

You implement each module starting from its "Start with" guidance:

```python
# search.py  --  minimal form
import os, re

pattern = re.compile(sys.argv[1])   # europa: CLI flag  ->  config
results = []
for root, dirs, files in os.walk("."):  # callisto: walk
    for f in files:
        if pattern.search(f):           # venus: filter
            results.append(os.path.join(root, f))
for r in results:                       # ganymede: display
    print(r)
```

You run it. It works. You commit.

### Phase 3: Evaluate  --  Test Against Failure Modes

You check each pattern's Known Failure Modes against your implementation:

| Pattern | Failure mode | Your tool |
|---------|-------------|-----------|
| callisto | Symlink loop | Not yet handled  --  you add a visited-set guard |
| venus | Predicate side effects | Safe  --  `re.search` is pure |
| ganymede | Format loss | Safe  --  plain text is lossless |

You also check [examined-projects.md](examined-projects.md) to see how `fd` handles similar issues: it uses `ignore::WalkParallel` for concurrent traversal with built-in symlink protection.

### Phase 4: Reflect  --  Check Splitting Triggers

No triggers have fired yet  --  your tool is 50 lines and meets all current needs. 5 whys analysis: each pattern's minimal form is still correct. The design is stable for this iteration.

### Next Iteration: Trigger Fires

**Trigger:** Users want `--exclude-dir node_modules`.

Open `callisto`  ->  "Split when you need exclusions." The pattern guides you to the next increment:

```python
def walk(root, should_skip=None):
    for dirpath, dirnames, filenames in os.walk(root):
        dirnames[:] = [d for d in dirnames if not should_skip(d)]
        yield from (os.path.join(dirpath, f) for f in filenames)
```

You commit and check: is the design stable now? The exclusion parameter is bounded, the change is backward-compatible, and the invariant (every reachable file visited once) still holds.

### Subsequent iterations

As more triggers fire, you iterate through the same loop:

| Trigger | Pattern | Next increment |
|---------|---------|---------------|
| Users want `--json` / `--csv` | **ganymede** | `display_json()`, `display_csv()` separated from plain output |
| Users want `*.md` / `*.py` flag filters | **venus** | Parse filter expressions from CLI args |
| Users pipe through stdin in varied encodings | **pluto** | Add input source abstraction, encoding detection |
| Path format tokens keep changing | **neptune** | Extract path formatter, unit-test independently |

After each increment, run Evaluate (check failure modes, compare against examined projects) and Reflect (5 whys on any issues). If all root causes are resolved across all patterns, the design is stable (`[DESIGN:STABLE]`). If not, loop back to Model.

---

## When to Ignore a Pattern

**Not every pattern applies to every project.** Some patterns only emerge at specific scales:

| Only at small scale | Only at large scale |
|--------------------|-------------------|
| venus (one-line filtering) | thebe (concurrent compaction) |
| ganymede (plain text output) | phoebe (resource lifecycle) |
| mars (JSON file save) | charon (durable workflow) |
| pluto (single `read()`) | (applies at all scales) |

A CLI tool with 500 users will not need `thebe`. A database engine with 500 users will. Size matters.

---

## Key Mindset Shifts

| If you think... | Instead, think... |
|---------------|------------------|
| "I need a design pattern." | "I have a problem -- which pattern describes it?" |
| "Let me pick the right abstraction upfront." | "Let me start minimal and observe when the naive form breaks." |
| "A router is an abstract dispatch mechanism." | "A router is a hash map from key to handler. Abstract only when it needs to be." |
| "I need undo/redo, so I need a transaction system." | "I need undo/redo, so I start with a single string and full-snapshot undo. I add deltas when memory pressure shows up." |
| "I should build a pipeline framework for my compiler." | "I should call three functions in sequence and check errors between them." |
| "I need a streaming parser because the input is large." | "I should read the whole file first and measure before I optimise." |
