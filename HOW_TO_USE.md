# How to Use This Repository

## The Problem

You're starting a new project. You have 25 problem patterns in front of you. Where do you begin?

## The Approach

**Don't read all 25 patterns.** The repository is designed for targeted lookup — not sequential reading.

---

## Step 1: Classify Your Project

Find your project type in the Quick Start table at the top of [INDEX.md](INDEX.md):

| Project type | Start here |
|--------------|-----------|
| CLI tool | venus, ganymede, callisto, europa |
| Web server / API | hyperion, saturn, neptune, europa |
| Editor / note app | pandora, mercury, mars, titan |
| Game | hyperion, saturn, mercury, mars, neptune |
| Embedded / RTOS | saturn, mars, hyperion, uranus |
| Compiler / interpreter | eris, neptune, jupiter, phoebe |
| Database / storage engine | mars, thebe, atlas, earth |
| Audio / video library | phoebe, neptune, hyperion, saturn |
| Workflow / orchestration | charon, mars, saturn, pan |
| Graphics / rendering | ganymede, mercury, neptune |

Read the "Start here" patterns. The "Watch for later" patterns typically emerge as the system grows.

---

## Step 2: Read the Patterns with Your Project in Mind

For each pattern, the **Practical Use** section gives actionable guidance. Read it in this order:

```
When this pattern matters      →  Does this describe my situation?
Signs you're in this pattern   →  Does my code smell like this?
Start with (minimal form)      →  What's the simplest implementation?
Split into its own module when →  When do I need to refactor?
Common traps                   →  What mistakes do people make?
Real project examples          →  Who else has solved this?
```

### What to look for

- **Not every "Start with" applies immediately.** Some patterns (like thebe/compaction, charon/workflow) describe problems that emerge at larger scales. Start with their minimal form only when you hit the first splitting trigger.
- **The "Start with" form is deliberately naive.** A JSON file, a single if-chain, a linear loop. The point is to prove correctness before layering abstraction.
- **The "Split when" triggers are the key moments.** They tell you when the naive form is about to break. Before that trigger fires, the naive form is the right answer.

---

## Step 3: Watch for Co-Occurring Patterns

The Common Clusters section in INDEX.md shows patterns that travel together:

```
saturn (route)   →  also look at hyperion (listener), jupiter (validate)
pandora (workpiece) →  also look at mercury (display), mars (persist)
eris (pipeline)  →  also look at neptune (transform) repeated N times
```

When you split a pattern into its own module, check its neighbors. If you're building a router, you almost certainly need validation and a listener too.

---

## Step 4: Recognize Splitting Boundaries

The AGENTS.md workflow (step 6–7) uses this repository as follows:

| You notice... | The pattern says... |
|--------------|-------------------|
| An if-chain grows to 15 branches | `saturn`: split when 20+ routes |
| Save/load is a single JSON dump | `mars`: split when incremental saves needed |
| Filtering becomes user-configurable | `venus`: split when predicates are dynamic |
| Multiple stages with shared context | `eris`: split when stages need independent testing |
| An edit application is full-string replace | `pandora`: split when large edits are slow |

The moment one of these triggers fires, open the corresponding pattern, read the "Split into its own module" section, and implement the next increment.

---

## Concrete Walkthrough: Building a CLI File Search Tool

### Step 1: Project classification

From the Quick Start table:

> **CLI tool** → Start here: **venus**, **ganymede**, **callisto**, **europa**

### Step 2: Read the patterns

You skim the four patterns:

| Pattern | Key guidance for your tool |
|---------|--------------------------|
| **venus** (filter) | Start with a simple `if` condition. Split when predicates become user-configurable. |
| **ganymede** (output format) | Start with hard-coded plain text. Split when user wants JSON/CSV flags. |
| **callisto** (filesystem walk) | Start with `os.Walk` / `glob`. Split when you need exclusions, depth limits. |
| **europa** (config assembly) | Start with CLI flags only. Split when you need config files + env vars. |

### Step 3: Implement the minimal version (~50 lines)

```python
# search.py — minimal form
import os, re

pattern = re.compile(sys.argv[1])   # europa: CLI flag → config
results = []
for root, dirs, files in os.walk("."):  # callisto: walk
    for f in files:
        if pattern.search(f):           # venus: filter
            results.append(os.path.join(root, f))
for r in results:                       # ganymede: display
    print(r)
```

You run it. It works. You commit.

### Step 4: Split when triggers fire

**Trigger 1:** Users want `--exclude-dir node_modules`.

Open `callisto` → "Split when you need exclusions." You add a `should_skip(dir)` parameter.

```python
def walk(root, should_skip=None):
    for dirpath, dirnames, filenames in os.walk(root):
        dirnames[:] = [d for d in dirnames if not should_skip(d)]
        yield from (os.path.join(dirpath, f) for f in filenames)
```

**Trigger 2:** Users want `--json` and `--csv` output.

Open `ganymede` → "Split when multiple output formats." You separate display logic.

```python
def display_json(results): ...
def display_csv(results): ...
def display_plain(results): ...  # original
```

**Trigger 3:** Users want `*.md` or `*.py` filter flags.

Open `venus` → "Split when predicates become dynamic." You parse filter expressions.

### Step 5: Check neighbors

From INDEX.md:
```
callisto (walk) → venus (filter), ganymede (output format)
```

You already have all three. The cluster is revealed — this is a natural grouping.

---

## When to Ignore a Pattern

**Not every pattern applies to every project.** Some patterns only emerge at specific scales:

| Only at small scale | Only at large scale |
|--------------------|-------------------|
| venus (one-line filter) | thebe (concurrent compaction) |
| ganymede (plain text output) | phoebe (resource lifecycle) |
| mars (JSON file save) | charon (durable workflow) |

A CLI tool with 500 users won't need `thebe`. A database engine with 500 users will. Size matters.

---

## How to Read a Full Pattern File

```
[Title + Pressure Class]  →  What kind of variation this pattern addresses
                            (Data / Behavior / Object / Rules)

Problem                   →  One-sentence definition
Context                   →  When does this arise?
Domains                   →  What pieces participate?
Shared Phenomena          →  What crosses the boundaries between pieces?
Invariants                →  What must always be true?
Failure Modes             →  What goes wrong?
Forces                    →  What tensions shape any solution?
Related Subproblems       →  What other patterns co-occur?
─────── PRACTICAL USE ────────
When it matters           →  Do you need this?
Signs you're in it        →  Recognition guide
Start with                →  Minimal form
Split when                →  Growth triggers
Common traps              →  Over-engineering patterns
Real examples             →  Examined projects that instantiate this
──────────────────────────────
Frame Concern             →  The fundamental difficulty
```

---

## Key Mindset Shifts

| If you think... | Instead, think... |
|---------------|------------------|
| "I need a design pattern." | "I have a problem — which pattern describes it?" |
| "Let me pick the right abstraction upfront." | "Let me start minimal and observe when the naive form breaks." |
| "A router is an abstract dispatch mechanism." | "A router is a hash map from key to handler. Abstract only when it needs to be." |
| "I need undo/redo, so I need a transaction system." | "I need undo/redo, so I start with a single string and full-snapshot undo. I add deltas when memory pressure shows up." |
| "I should build a pipeline framework for my compiler." | "I should call three functions in sequence and check errors between them." |
