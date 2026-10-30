# Pattern Index

See [how-to-use.md](how-to-use.md) for a project-type-to-patterns reference.

## Application patterns

| Planet | Pattern | File |
|---|---|---|
| callisto | filesystem walk | [callisto-filesystem-walk.md](application-patterns/callisto-filesystem-walk.md) |
| earth | sorting | [earth-sorting.md](application-patterns/earth-sorting.md) |
| eris | pipeline | [eris-pipeline.md](application-patterns/eris-pipeline.md) |
| europa | configuration assembly | [europa-config-assembly.md](application-patterns/europa-config-assembly.md) |
| ganymede | output formatting | [ganymede-output-format.md](application-patterns/ganymede-output-format.md) |
| hyperion | network listener | [hyperion-network-listener.md](application-patterns/hyperion-network-listener.md) |
| iota | command execution | [iota-command-execution.md](application-patterns/iota-command-execution.md) |
| janus | reduction / aggregation | [janus-reduction-aggregation.md](application-patterns/janus-reduction-aggregation.md) |
| jupiter | validation | [jupiter-validation.md](application-patterns/jupiter-validation.md) |
| mars | persistence | [mars-persistence.md](application-patterns/mars-persistence.md) |
| mercury | display | [mercury-display.md](application-patterns/mercury-display.md) |
| neptune | transformation | [neptune-transformation.md](application-patterns/neptune-transformation.md) |
| pandora | simple workpieces | [pandora-simple-workpieces.md](application-patterns/pandora-simple-workpieces.md) |
| phoebe | resource lifecycle | [phoebe-resource-lifecycle.md](application-patterns/phoebe-resource-lifecycle.md) |
| pluto | input parsing | [pluto-input-parsing.md](application-patterns/pluto-input-parsing.md) |
| rhea | syntax detection | [rhea-syntax-detection.md](application-patterns/rhea-syntax-detection.md) |
| saturn | routing | [saturn-routing.md](application-patterns/saturn-routing.md) |
| titan | syntax highlighting | [titan-syntax-highlighting.md](application-patterns/titan-syntax-highlighting.md) |
| uranus | authentication | [uranus-authentication.md](application-patterns/uranus-authentication.md) |
| venus | filtering | [venus-filtering.md](application-patterns/venus-filtering.md) |

## System patterns

| Planet | Pattern | File |
|---|---|---|
| atlas | memory allocation | [atlas-memory-allocation.md](system-patterns/atlas-memory-allocation.md) |
| charon | durable workflow | [charon-durable-workflow.md](system-patterns/charon-durable-workflow.md) |
| dione | power management | [dione-power-management.md](system-patterns/dione-power-management.md) |
| leda | feedback control | [leda-feedback-control.md](system-patterns/leda-feedback-control.md) |
| pan | resource scheduling | [pan-resource-scheduling.md](system-patterns/pan-resource-scheduling.md) |
| thebe | background compaction | [thebe-background-compaction.md](system-patterns/thebe-background-compaction.md) |

## Common Clusters

| Primary pattern | Often accompanied by |
|---|---|
| saturn (routing) | hyperion (listener), jupiter (validation) |
| pandora (workpiece) | mercury (display), mars (persistence), neptune (transformation) |
| callisto (walk) | venus (filtering), ganymede (output format) |
| hyperion (listener) | neptune (transformation), saturn (routing) |
| titan (highlighting) | rhea (detection) |
| eris (pipeline) | neptune (transformation) repeated N times |
| pluto (parsing) | jupiter (validation), neptune (transformation), mars (persistence) |
| phoebe (lifecycle) | mars (persistence), hyperion (listener), atlas (memory) |
| thebe (compaction) | mars (persistence), pan (scheduling), earth (sorting) |
| charon (workflow) | mars (persistence), saturn (routing), iota (command execution) |
| janus (reduction) | venus (filtering), neptune (transformation), earth (sorting), eris (pipeline) |

## Refactoring Catalog

[refactoring/refactoring-catalog.md](refactoring/refactoring-catalog.md) maps co-change cluster signals to concrete refactoring techniques.
