# ------------------------------------------------------------------------------
#  phoebe-resource-lifecycle.md
#  Object variation -- the resource type changes while the acquire/release contract stays the same.
# ------------------------------------------------------------------------------

# - Problem -

Your system manages a finite resource -- an audio device, file handle, GPU context, network socket, or memory arena. It must be acquired before use, released after use, and guaranteed to be released on every exit path, including errors and early returns.

# - When You'll See This -

Any system that manages limited resources with global side effects. Audio libraries (init/uninit), GPU programming (context creation/destruction), file I/O (open/close), network services (bind/close), memory management (arena allocate/reset). The lifecycle is visible through explicit init/uninit or open/close function pairs.

# - How It Works -

- **Resource**: the finite entity being managed -- audio device, file handle, GPU device, network socket.
- **Lifecycle states**: uninitialized -> initializing -> ready -> error -> closing -> closed.
- **Acquire**: obtain the resource and transition it to the ready state.
- **Release**: return the resource to the system and transition to closed.
- **Guard**: a mechanism (RAII in C++, `with` in Python, `defer` in Go) that ensures release happens on all exit paths.

# - What Must Always Be True -

- Every acquire has a matching release on all exit paths (success, error, early return, exception).
- Acquiring an already-acquired resource either fails or is a no-op (no double-init).
- Releasing an already-released resource either fails or is a no-op (no double-free).
- Partial initialisation (an error occurs mid-init) leaves no resources leaked.
- Concurrent access to the same resource is safe (mutex, atomic state, or single-owner semantics).

# - What Goes Wrong -

- **Resource leak**: an acquire without a matching release eventually exhausts the resource pool.
- **Double-free**: releasing the same resource twice corrupts the allocator's state.
- **Use-after-free**: using a resource after it's been released causes undefined behaviour.
- **Leak on error path**: an early return or exception bypasses the release call.
- **Thread safety**: two threads access the same resource without synchronisation.
- **Partial init leak**: initialisation fails halfway through, and the partially-allocated sub-resources are never freed.

# - Design Trade-Offs -

- **Automatic vs. manual management**: RAII/defer/`with` ensures release automatically but constrains lifetimes; manual open/close gives flexibility but risks leaks.
- **Eager vs. lazy acquisition**: acquire at startup (fail fast, simple) vs. acquire on demand (efficient, but can fail mid-operation).
- **Single-owner vs. shared**: single ownership is simple and safe; shared ownership (reference counting) is flexible but adds complexity and can leak with cycles.

# - Related Problems -

- **atlas/memory-allocation**: memory is a resource with an acquire/release lifecycle.
- **hyperion/network-listener**: sockets are resources with open/close lifecycles.
- **mars/persistence**: file handles are resources with open/close lifecycles.

# - Getting Started -

Pair every `open()` with a `close()` in the same function. Use the language's resource management: Python's `with` statement, C++ RAII, Rust's ownership, Go's `defer`. One resource, no sharing.

# - When to Split Into Its Own Module -

- You manage multiple resources with different lifecycle requirements.
- Resources need to be shared across threads.
- You need to track resource usage for debugging or profiling.
- The resource lifecycle logic is duplicated across the codebase.

# - Signs You're in This Pattern -

- You have init/uninit or open/close function pairs.
- You need to ensure cleanup happens on error paths.
- Resource leaks are a recurring bug in your system.
- You have a pool or cache of limited resources.

# - Common Mistakes -

- Not using the language's resource management features (RAII, `with`, `defer`).
- Manually calling close on every exit path instead of using a guard -- one early return misses it and leaks.
- Allowing resources to be used after they've been released.
- Not protecting shared resources with synchronisation.

# - The Core Challenge -

Resource management is a correctness problem on every exit path. A leak looks like no problem during testing (resources are plentiful) but becomes a crash in production (resource exhaustion). The challenge is ensuring that every acquire has exactly one matching release, on every code path, forever.
