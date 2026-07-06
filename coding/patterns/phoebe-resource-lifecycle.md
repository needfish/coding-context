# phoebe -- resource lifecycle

## Problem

A resource (file handle, network socket, database connection, mutex, memory buffer) must be acquired before use and released after use on every code path: normal return, early return, error, signal. If release is missed, the resource leaks.

## Input

- An acquire operation that returns a resource handle
- A release operation that takes a resource handle and frees it

## Output

- Acquire: a resource handle (or an error if the resource cannot be acquired)
- Release: nothing (the resource is freed)

## Observable Behavior

- Every acquired resource is released exactly once
- A resource is never used after it is released
- If the resource cannot be acquired, the caller receives a clear error and no handle
- Resources are released even when an error occurs during use
- Releasing a resource that was already released causes undefined behaviour (crash, corruption)

## Minimal Code

```python
# Python: context manager (RAII equivalent)
with open("file.txt") as f:
    data = f.read()
# File is automatically closed when the 'with' block exits
```

```rust
// Rust: RAII
let f = File::open("file.txt")?;
// f is automatically closed when it goes out of scope
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Resource leak | Number of open handles (file descriptors, connections) grows until the system runs out | Use RAII (C++), with (Python), try-with-resources (Java), defer (Go) |
| Double release | Freeing a resource that is already freed causes a crash | Track whether the resource has been released (boolean flag, Option type) |
| Use after release | Using a freed handle reads stale data or crashes | Invalidate the handle on release (set to null, use a generation counter) |
| Stale resource | Remote end closed the connection but local code still holds the handle | Heartbeat/health check before use |
| Pool exhaustion | Resource pool is empty because acquired resources were not returned | Enforce a maximum lease duration and forcibly reclaim leaked resources |

## Design Trade-offs

| Choice | What changes |
|---|---|
| RAII vs. explicit acquire/release | RAII (constructor acquires, destructor releases) is safe but ties resource lifetime to a scope. Explicit acquire/release is flexible but error-prone. |
| Pooled vs. per-use allocation | Pooling avoids allocation overhead but needs borrowing/returning logic. Per-use allocation is simpler but slower. |
| Eager vs. lazy initialisation | Eager (acquire at startup) ensures resources are available but wastes them if unused. Lazy (acquire on first use) saves resources but adds latency. |
| Blocking vs. non-blocking acquire | Blocking (wait until available) is simple but blocks the caller. Non-blocking (fail immediately) requires the caller to handle unavailability. |

## Start With

Use the language's built-in resource management. In Python, use `with` statements. In Rust, let RAII handle it. In Go, use `defer`. In C, manually pair every `malloc` with `free` and every `fopen` with `fclose`.

## Split Into Its Own Module When

- More than two resource types need lifecycle management.
- Resources need to be pooled (database connections, thread pools).
- Resource lifecycle needs to be logged or monitored.
- Resources require health checking or automatic reconnection.

## Signs This Is Present

- Resources are opened, connected, or allocated and must be closed, disconnected, or freed.
- try/finally, defer, or context managers are used.
- A connection pool or buffer pool is implemented.
- Resource cleanup logic is duplicated across multiple functions.

## Common Mistakes

- Relying on finalisers/GC for resource cleanup (non-deterministic timing, may not run).
- Not releasing resources in all error paths (early return, exception, signal).
- Holding a resource longer than needed.
- Using a resource after releasing it (dangling handle).
- Not handling the case where resource acquisition itself fails.
- Creating a resource but not binding it to a scope.
