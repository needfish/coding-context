# Pattern Index

See [how-to-use.md](how-to-use.md) for a project-type-to-patterns reference.

## Pattern List

| Planet | Pattern | File |
|---|---|---|
| mercury | display | [patterns/mercury-display.md](patterns/mercury-display.md) |
| venus | filtering | [patterns/venus-filtering.md](patterns/venus-filtering.md) |
| earth | sorting | [patterns/earth-sorting.md](patterns/earth-sorting.md) |
| mars | persistence | [patterns/mars-persistence.md](patterns/mars-persistence.md) |
| jupiter | validation | [patterns/jupiter-validation.md](patterns/jupiter-validation.md) |
| saturn | routing | [patterns/saturn-routing.md](patterns/saturn-routing.md) |
| uranus | authentication | [patterns/uranus-authentication.md](patterns/uranus-authentication.md) |
| neptune | transformation | [patterns/neptune-transformation.md](patterns/neptune-transformation.md) |
| callisto | filesystem walk | [patterns/callisto-filesystem-walk.md](patterns/callisto-filesystem-walk.md) |
| iota | command execution | [patterns/iota-command-execution.md](patterns/iota-command-execution.md) |
| ganymede | output formatting | [patterns/ganymede-output-format.md](patterns/ganymede-output-format.md) |
| europa | configuration assembly | [patterns/europa-config-assembly.md](patterns/europa-config-assembly.md) |
| titan | syntax highlighting | [patterns/titan-syntax-highlighting.md](patterns/titan-syntax-highlighting.md) |
| rhea | syntax detection | [patterns/rhea-syntax-detection.md](patterns/rhea-syntax-detection.md) |
| janus | reduction / aggregation | [patterns/janus-reduction-aggregation.md](patterns/janus-reduction-aggregation.md) |
| hyperion | network listener | [patterns/hyperion-network-listener.md](patterns/hyperion-network-listener.md) |
| pan | resource scheduling | [patterns/pan-resource-scheduling.md](patterns/pan-resource-scheduling.md) |
| atlas | memory allocation | [patterns/atlas-memory-allocation.md](patterns/atlas-memory-allocation.md) |
| dione | power management | [patterns/dione-power-management.md](patterns/dione-power-management.md) |
| leda | feedback control | [patterns/leda-feedback-control.md](patterns/leda-feedback-control.md) |
| pandora | simple workpieces | [patterns/pandora-simple-workpieces.md](patterns/pandora-simple-workpieces.md) |
| eris | pipeline | [patterns/eris-pipeline.md](patterns/eris-pipeline.md) |
| phoebe | resource lifecycle | [patterns/phoebe-resource-lifecycle.md](patterns/phoebe-resource-lifecycle.md) |
| pluto | input parsing | [patterns/pluto-input-parsing.md](patterns/pluto-input-parsing.md) |
| thebe | background compaction | [patterns/thebe-background-compaction.md](patterns/thebe-background-compaction.md) |
| charon | durable workflow | [patterns/charon-durable-workflow.md](patterns/charon-durable-workflow.md) |

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
