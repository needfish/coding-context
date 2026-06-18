# phoebe: Resource Lifecycle

> **Pressure class:** Object variation — the resource type changes while the acquire/release contract stays the same.

## Problem

A finite system resource (audio device, file handle, GPU context, network socket, memory arena) must be acquired before use, released after use, and guaranteed to be released on every exit path — including error paths, early returns, and exceptions.

## Context

Arises whenever a system manages resources that are limited in number, have global side effects, or must be returned to a known state after use. Found in audio libraries (init/uninit), GPU programming (context creation/destruction), file I/O (open/close), network services (bind/close), and memory management (arena allocate/reset). The lifecycle is visible to the caller through explicit init/uninit or open/close function pairs. The system must ensure that partial initialisation does not leak resources and that concurrent access to the same resource is safe.

## Domains Involved

- **Resource** (causal): the finite entity being managed (audio device, file handle, GPU device, network socket). Has state: uninitialized, initializing, ready, error, closing.
- **Resource handle** (lexical): an opaque identifier or pointer through which the caller interacts with the resource.
- **Acquirer** (causal): the caller that requests the resource and is responsible for releasing it.
- **Initialiser** (causal): the system component that sets up the resource, allocates backing state, and transitions it to the ready state.
- **Finaliser** (causal): the system component that tears down the resource, releases backing state, and transitions it to the uninitialized state.
- **Configuration** (lexical): the parameters that control how the resource is initialised (device ID, buffer size, format, flags).

## Shared Phenomena

- Acquirer → Initialiser: configuration parameters
- Initialiser → Resource: allocation, configuration, state transition
- Initialiser → Acquirer: ready resource handle (or error)
- Acquirer → Resource: usage operations (read, write, process)
- Acquirer → Finaliser: release request
- Finaliser → Resource: deallocation, state cleanup
- Initialiser → Finaliser: partial initialisation state (for cleanup on error during init)

## Invariants

- A resource must not be used before it is initialised or after it is finalised.
- Initialisation must be atomic from the caller's perspective — either the resource is fully ready, or initialisation fails and no resource is leaked.
- Finalisation must succeed even if some operations on the resource failed during its lifetime.
- A resource must not be finalised while it is in use by a concurrent operation.
- Calling init on an already-initialised resource must either fail gracefully or reinitialise it (defined policy).
- Calling uninit on an already-uninitialised resource must be safe (idempotent or no-op).

## Known Failure Modes

- **Partial init leak**: Initialisation allocates resource A successfully, then fails on resource B. The destructor is never called, and resource A leaks.
- **Double free**: The caller calls uninit twice, and the second call operates on freed memory or closed handles.
- **Use-after-free**: A concurrent operation holds a reference to the resource after the caller has finalised it.
- **Init-order dependency**: Resource A must be initialised before resource B, but the caller initialises B first and A fails, leaving B orphaned.
- **Error swallowed during finalise**: The finaliser encounters an error (e.g., close() returns EIO) but the caller has no way to handle it because the resource is already logically released.
- **Resource exhaustion**: Resources are acquired but not released (even one leak per call in a long-running process exhausts the pool).
- **Context manager mismatch**: The caller uses the wrong finaliser for a resource (e.g., closes a device handle instead of releasing an audio engine).

## Forces

- **Manual vs. automatic lifecycle**: Manual init/uninit gives the caller full control over lifetime but is error-prone; automatic lifecycle (RAII, GC, reference counting) is safer but introduces unpredictability in release timing.
- **Synchronous vs. asynchronous finalisation**: Synchronous finalisation is simple but blocks the caller; asynchronous finalisation (deferred cleanup) is non-blocking but complicates resource state tracking.
- **Leak detection vs. performance**: Tracking every allocation for leak detection adds memory and CPU overhead; skipping tracking risks undetected leaks in production.
- **Implicit vs. explicit acquisition**: Some resources are acquired implicitly (first use) and released implicitly (last use); explicit acquire/release is clearer but requires caller discipline.

## Related Subproblems

- **mars/persist**: File handles and storage resources follow the lifecycle pattern.
- **hyperion/network-listener**: Network sockets must be opened, configured, and closed through a lifecycle.
- **atlas/memory-allocation**: Memory arenas have explicit init/reset/destroy lifecycles.
- **pan/resource-scheduling**: Resource acquisition may block waiting for a contended resource.
- **jupiter/validate**: Initialisation parameters must be validated before resource creation.

### ==== Practical Use ====

#### When this pattern matters
Any project that manages external resources — audio devices, GPU contexts, file handles, network sockets, hardware peripherals, memory arenas.

#### Start with (minimal form)
A single `init()` function that allocates resources in order, and a single `uninit()` function that frees them in reverse order. Use a boolean flag to track initialised state. No error recovery during init beyond returning an error code.

```c
typedef struct { bool ready; ... } MyResource;
int my_init(MyResource *r, Config *cfg) {
    if (r->ready) return -1;
    // allocate resources in order
    if (allocate_a() < 0) return -1;
    if (allocate_b() < 0) { free_a(); return -1; }
    r->ready = true;
    return 0;
}
void my_uninit(MyResource *r) {
    if (!r->ready) return;
    // free in reverse order
    free_b(); free_a();
    r->ready = false;
}
```

#### Split into its own module when (observable triggers)
- Initialisation involves more than 3 resources that must be cleaned up independently on failure.
- The resource has multiple valid states (uninitialised, initialising, ready, suspended, error) with different allowed operations per state.
- Resources have dependencies on each other (A must outlive B).
- Concurrent access to the same resource is possible (reference counting or locking needed).
- The caller needs to configure the resource differently across invocations.

#### Common over-engineering traps
- Adding reference counting before confirming concurrent access is a real problem.
- Implementing a full state machine when a boolean `ready` flag suffices.
- Building an automatic resource manager (RAII wrapper) before manual init/uninit causes measurable bugs.
- Adding asynchronous initialisation before measuring that synchronous init blocks the caller.

## Frame Concern

**Leak prevention across all exit paths.** A resource that is acquired must be released exactly once, regardless of how many early returns, error branches, or exception paths exist between acquire and release. The concern is not about acquiring the resource but about guaranteeing its release under every possible execution path — a control-flow problem, not a resource problem.
