# eris -- pipeline

## Problem

Data must pass through a sequence of transformation steps, where each step feeds into the next. The number and kind of steps change over time as the system evolves. Examples: tokenise -> parse -> typecheck -> codegen, or load -> validate -> transform -> save.

## Input

- Raw data (any type)
- A list of stage functions, each taking one value and returning another

## Output

- The result after all stages have been applied in order

## Observable Behavior

- Stage 1 receives the input and returns a result
- Stage 2 receives stage 1's result and returns a result
- This continues through all stages in order
- If any stage fails (raises an error), the pipeline stops at that point
- Each stage is independent: it does not look at data from stages before its immediate predecessor
- The same input and stages always produce the same output

## Minimal Code

```python
def pipeline(input_data, *stages):
    result = input_data
    for stage in stages:
        result = stage(result)
    return result

# Usage:
result = pipeline(data, parse, validate, transform, format)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Stage coupling | A stage reads data from two stages before, creating hidden dependency if stages are reordered | Each stage must only use its immediate input; pass shared data through a context object |
| Error swallowing | A stage catches all errors and returns a default, hiding failures | Propagate errors to the pipeline error handler |
| Pipeline too long | 20+ stages become hard to read and debug | Group related stages into sub-pipelines |
| Context bloat | The shared context object accumulates every field any stage might need | Split context into focused sub-objects per stage group |
| Global state in stages | A stage uses a global variable instead of the context, making tests order-dependent | Pass dependencies explicitly through the pipeline |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Explicit vs. implicit piping | Explicit (stage1(stage2(x))) is simple but unreadable with many stages. Implicit (pipeline.add(stage1).add(stage2).run(x)) is readable but adds infrastructure. |
| Eager vs. lazy evaluation | Eager processes the entire input at once, memory proportional to input size. Lazy (generators) uses memory proportional to the stage window. |
| Sequential vs. parallel | Sequential is simple but uses one core. Parallel (within stages or via queues) improves throughput but adds concurrency bugs. |

## Start With

The minimal code above. Call functions in sequence using a for loop. Add a pipeline abstraction when there are more than 5 stages or stages need to be added/removed at runtime.

## Split Into Its Own Module When

- The pipeline has more than 5 stages.
- Stages need to be added, removed, or reordered at runtime.
- Error handling per stage is needed (retry, skip, fail-fast).
- Individual stages need independent testing with fixtures.
- Parallel stage execution is needed for performance.

## Signs This Is Present

- Functions are composed in a chain: f(g(h(x))).
- A list of transformations is applied iteratively to data.
- Middleware or plugin architecture is used.
- A chain of responsibility pattern is present.

## Common Mistakes

- Making stages stateful (shared mutable state between stages causes ordering bugs).
- Creating implicit dependencies between non-adjacent stages.
- Using a pipeline framework before the pipeline has 3+ stages.
- Ignoring backpressure in streaming pipelines (fast producer, slow consumer).
- Mixing error handling strategies without explicit documentation.
