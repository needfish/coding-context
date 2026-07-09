# context files
- contains general guides I feed to coding agents in my daily use, researched after a year of reading about software engineering papers.
- includes a pretty weird attempts at mapping scenarios/patterns of code, to help with modularizing.


```
.
|-- AGENTS.md                  # Agent instructions (entry point)
|-- README.md                  # This file
|-- .gitignore
|-- answering/                 # Answering style examples
|   +-- msdn-example-2.md      # there's MS's Raymond Chen's posts here that I would like the agent to copy, but it conflicts    with the no first person rule
+-- coding/                    # Coding conventions and patterns
    |-- README.md
    |-- conventions/            # Code style conventions
    |   |-- comments.md
    |   |-- correctness-reasoning.md
    |   |-- implementation.md
    |   |-- layout.md
    |   +-- naming.md
    |-- error-handling/
    |   +-- error-handling.md
    |-- examined-projects.md
    |-- heuristics/
    |   +-- heuristics.md
    |-- how-to-use.md
    |-- modeling/               # Data and domain modeling
    |   |-- invariants.md
    |   |-- many-forms.md
    |   |-- modeling-communication.md
    |   |-- operation-invariants.md
    |   |-- optionality.md
    |   |-- process.md
    |   |-- state-machine.md
    |   +-- type-precision.md
    |-- pattern-table.md
    |-- patterns/               # Named implementation patterns
    |   |-- atlas-memory-allocation.md
    |   |-- callisto-filesystem-walk.md
    |   |-- charon-durable-workflow.md
    |   |-- dione-power-management.md
    |   |-- earth-sorting.md
    |   |-- eris-pipeline.md
    |   |-- europa-config-assembly.md
    |   |-- ganymede-output-format.md
    |   |-- hyperion-network-listener.md
    |   |-- io-command-execution.md
    |   |-- janus-reduction-aggregation.md
    |   |-- jupiter-validation.md
    |   |-- leda-feedback-control.md
    |   |-- mars-persistence.md
    |   |-- mercury-display.md
    |   |-- neptune-transformation.md
    |   |-- pan-resource-scheduling.md
    |   |-- pandora-simple-workpieces.md
    |   |-- phoebe-resource-lifecycle.md
    |   |-- pluto-input-parsing.md
    |   |-- rhea-syntax-detection.md
    |   |-- saturn-routing.md
    |   |-- thebe-background-compaction.md
    |   |-- titan-syntax-highlighting.md
    |   |-- uranus-authentication.md
    |   +-- venus-filtering.md
    |-- project-conventions/
    |   +-- project-conventions.md
    |-- refactoring/
    |   |-- defunctionalization.md
    |   |-- refactoring-catalog.md
    |   +-- reflect.md
    |-- testing/
    |   +-- testing.md
    +-- toolchain/
        |-- safety-tools.md
        |-- toolchain-defaults.md
        +-- web-stack.md
```
