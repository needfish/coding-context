# Reflection Process

## Document Failed Assumptions

List each assumption that failed, the evidence that disproved it, and the correction applied.

## Check Splitting Triggers

Open each pattern's "Split into its own module when" and "Common traps" sections from [patterns/](../patterns/). Check if any splitting triggers have fired.

## File-Level Splitting

For file-level splitting and merging, follow Evan Czaplicki's "The Life of a File" strategy: start in one file, extract only when a coherent abstraction emerges, and split when a module has multiple responsibilities that change independently.

## Code Transformations

For within-file code transformations (extract method, inline class, algebraic to abstract type, a-to-b transform), open [refactoring-catalog.md](refactoring-catalog.md). Identify which paradigm the module uses (object-oriented, functional/algebraic, or data-oriented). Find the co-change signal in the matching map, then follow the linked refactoring technique.

## Explore Adjacent Domains

If stuck, read the pattern catalog's Adjacent Domains section. List 3 domains, extract 3-5 structural elements (architecture, operations, failure modes) from each, test one fit with a tiny example.

## Negative Knowledge

Document negative knowledge -- what did not work and why. Share it with the user.

## Git Co-Change Analysis

Run `git log --oneline --name-only --pretty=format: --since=<last-design-stable>` to extract files that changed together in the same commit. Cluster files by co-occurrence count. Compare clusters against the current module structure.

- If a cluster spans multiple modules, merge or refactor so co-changing files share a single module (Common Closure Principle).
- If files in the same module never co-change, split them into separate modules (Common Reuse Principle).
- Ignore clusters with fewer than 3 co-occurrences or 5 files.
- After each refactoring, re-run the analysis to confirm containment.

Output the co-change clusters found, the violations flagged, and the refactoring applied.

## 5 Whys Analysis

Perform 5 whys on every issue. For each symptom, drill down: Why did it happen? -> Why was that the case? -> (repeat five times). At the fifth why the root cause is reached. If already addressed in the design, mark resolved. If not, the design needs another iteration.

## Termination

Output `[DESIGN:STABLE]` when ALL root causes resolved and all co-change clusters contained within single modules.
Output `[PHASE:4:DONE]` when unresolved root causes remain and another iteration is required.
