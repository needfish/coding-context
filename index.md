# Pattern Index

See [how-to-use.md](how-to-use.md) for a project-type-to-patterns reference and a walk-through of how to use this repository.

---

| Planet | Pattern | Pressure Class | File | Frame Concern |
|--------|---------|---------------|------|---------------|
| mercury | display | Data variation | [patterns/mercury-display.md](patterns/mercury-display.md) | Temporal coupling between source and display |
| venus | filtering | Object variation | [patterns/venus-filtering.md](patterns/venus-filtering.md) | Predicate ownership |
| earth | sorting | Behavior variation | [patterns/earth-sorting.md](patterns/earth-sorting.md) | Comparator consistency |
| mars | persistence | Object variation | [patterns/mars-persistence.md](patterns/mars-persistence.md) | Durability guarantee boundary |
| jupiter | validation | Rules variation | [patterns/jupiter-validation.md](patterns/jupiter-validation.md) | Rule set completeness |
| saturn | routing | Behavior variation | [patterns/saturn-routing.md](patterns/saturn-routing.md) | Route table consistency |
| uranus | authentication | Object variation | [patterns/uranus-authentication.md](patterns/uranus-authentication.md) | Trust delegation |
| neptune | transformation | Behavior variation | [patterns/neptune-transformation.md](patterns/neptune-transformation.md) | Semantic equivalence across heterogeneous representations |
| callisto | filesystem walk | Data variation | [patterns/callisto-filesystem-walk.md](patterns/callisto-filesystem-walk.md) | Gap between filesystem order and consumer order |
| io | command execution | Object variation | [patterns/io-command-execution.md](patterns/io-command-execution.md) | Subprocess lifecycle vs. consumer expectations |
| ganymede | output formatting | Data variation | [patterns/ganymede-output-format.md](patterns/ganymede-output-format.md) | Terminal capability detection |
| europa | configuration assembly | Data variation | [patterns/europa-config-assembly.md](patterns/europa-config-assembly.md) | Merge ordering transparency |
| titan | syntax highlighting | Behavior variation | [patterns/titan-syntax-highlighting.md](patterns/titan-syntax-highlighting.md) | Grammar coverage vs. source variety |
| rhea | syntax detection | Behavior variation | [patterns/rhea-syntax-detection.md](patterns/rhea-syntax-detection.md) | Information deficit at decision time |
| janus | reduction / aggregation | Behavior variation | [patterns/janus-reduction-aggregation.md](patterns/janus-reduction-aggregation.md) | Associativity and the identity element |
| iapetus | code counting | Behavior variation | [patterns/iapetus-code-counting.md](patterns/iapetus-code-counting.md) | Grammar coverage vs. real-world source variety |
| hyperion | network listener | Object variation | [patterns/hyperion-network-listener.md](patterns/hyperion-network-listener.md) | Connection establishment vs. request processing gap |
| pan | resource scheduling | Behavior variation | [patterns/pan-resource-scheduling.md](patterns/pan-resource-scheduling.md) | Priority assignment vs. actual timeliness |
| atlas | memory allocation | Object variation | [patterns/atlas-memory-allocation.md](patterns/atlas-memory-allocation.md) | Request size granularity vs. page granularity |
| dione | power management | Behavior variation | [patterns/dione-power-management.md](patterns/dione-power-management.md) | Energy consumption vs. energy availability |
| leda | feedback control | Behavior variation | [patterns/leda-feedback-control.md](patterns/leda-feedback-control.md) | Measurement noise vs. control stability |
| pandora | simple workpieces | Behavior variation | [patterns/pandora-simple-workpieces.md](patterns/pandora-simple-workpieces.md) | Edit operations vs. document state consistency |
| eris | pipeline | Behavior variation | [patterns/eris-pipeline.md](patterns/eris-pipeline.md) | Stage coupling across consecutive transforms |
| phoebe | resource lifecycle | Object variation | [patterns/phoebe-resource-lifecycle.md](patterns/phoebe-resource-lifecycle.md) | Leak prevention across all exit paths |
| pluto | input parsing | Behavior variation | [patterns/pluto-input-parsing.md](patterns/pluto-input-parsing.md) | Gap between byte sequences and structured semantics |
| thebe | background compaction | Behavior variation | [patterns/thebe-background-compaction.md](patterns/thebe-background-compaction.md) | Consistency during concurrent storage reorganisation |
| charon | durable workflow | Behavior variation | [patterns/charon-durable-workflow.md](patterns/charon-durable-workflow.md) | Progress across failure boundaries |

### ==== Common Clusters ====

These patterns co-occur so frequently that seeing one should make you
look for the others. During the **Model** phase, when a pattern's Related
Subproblems reference another pattern, check whether your system also
instantiates that second problem type:

| Primary pattern | Often accompanied by |
|----------------|---------------------|
| saturn (routing) | hyperion (listener), jupiter (validation) |
| pandora (workpiece) | mercury (display), mars (persistence), neptune (transformation) |
| callisto (walk) | venus (filtering), ganymede (output format) |
| hyperion (listener) | neptune (transformation), saturn (routing) |
| titan (highlighting) | rhea (detection), iapetus (counting) |
| eris (pipeline) | neptune (transformation) repeated N times |
| pluto (parsing) | jupiter (validation), neptune (transformation), mars (persistence) |
| phoebe (lifecycle) | mars (persistence), hyperion (listener), atlas (memory) |
| thebe (compaction) | mars (persistence), pan (scheduling), earth (sorting) |
| charon (workflow) | mars (persistence), saturn (routing), io (command execution) |
| janus (reduction) | venus (filtering), neptune (transformation), earth (sorting), eris (pipeline) |

## Refactoring Catalog

[refactoring-catalog.md](refactoring-catalog.md) maps co-change cluster signals
(found during Reflect phase git history analysis) to concrete refactoring
techniques from three catalogs:

