# ------------------------------------------------------------------------------
#  atlas-memory-allocation.md
#  Object variation -- the allocation strategy changes while the allocate/free contract stays the same.
# ------------------------------------------------------------------------------

# - Problem -

Your program needs to carve up a pool of memory into variable-sized blocks for different requests, then recycle those blocks when they're freed -- all while avoiding fragmentation and keeping it fast.

# - When You'll See This -

Any system that manages dynamic memory. General-purpose allocators (malloc), heap implementations for embedded systems, slab allocators for kernel objects, pool allocators for game engines, and garbage-collected heaps. Different workloads demand different strategies.

# - How It Works -

- **Memory pool**: the contiguous or segmented region of memory the allocator manages.
- **Allocation request**: a call asking for N bytes of memory.
- **Free request**: a call returning previously allocated memory to the pool.
- **Free list**: the data structure tracking which blocks are available.
- **Metadata**: bookkeeping information embedded in each block (size, next/prev pointers, flags).

# - What Must Always Be True -

- Every allocation returns a region of at least the requested size that doesn't overlap with any other live allocation.
- Freeing a valid pointer returns that memory to the pool for reuse.
- Double-free or use-after-free produces undefined behaviour -- the allocator can detect some cases but not all.
- Metadata overhead is bounded and predictable.
- The allocator handles concurrent access correctly (locking or lock-free).

# - What Goes Wrong -

- **Fragmentation**: after many allocations and frees, the pool has many small gaps and no single region large enough for a new request, even though total free space is sufficient.
- **Memory leak**: allocated blocks are never freed, exhausting the pool over time.
- **Fragmentation**: external fragmentation (gaps between blocks) and internal fragmentation (wasted space inside a block) both waste memory.
- **False sharing**: two threads write to different data that happens to share a cache line, causing performance degradation.
- **Blowup**: per-allocation metadata overhead becomes significant for many small allocations.

# - Design Trade-Offs -

- **Speed vs. fragmentation**: a simple free list (first-fit) is fast but fragments; a buddy allocator is slower but fragments less.
- **Generality vs. specialisation**: a general-purpose allocator handles any size but is slower for specific patterns; a pool allocator is fast for fixed-size objects but can't handle variable sizes.
- **Thread-safe vs. single-threaded**: locking every allocation is safe but slow; thread-local caches are fast but risk imbalance.

# - Related Problems -

- **dione/power-management**: memory allocation and deallocation has energy cost.
- **phoebe/resource-lifecycle**: memory is a resource with acquire/release lifecycle.

# - Getting Started -

Use the language's built-in allocator. In C, `malloc` / `free`. In Rust, the default allocator. No custom allocator, no pool, no slab.

# - When to Split Into Its Own Module -

- You need to allocate many objects of the same size (pool allocator helps).
- You need to bound allocation latency (real-time requirements).
- The default allocator causes unacceptable fragmentation in your workload.
- You need to track allocation statistics for debugging or profiling.

# - Signs You're in This Pattern -

- You're considering a custom memory allocator.
- Your application uses lots of small, short-lived allocations.
- Memory fragmentation causes OOM errors despite enough total free memory.
- You need deterministic allocation latency.

# - Common Mistakes -

- Writing a custom allocator before profiling shows the default one is a problem.
- Using a general-purpose allocator for fixed-size objects (pool allocator is simpler and faster).
- Ignoring alignment requirements of the platform.
- Not handling OOM gracefully.

# - The Core Challenge -

Allocation is a search problem -- finding a free block of at least the requested size. The search strategy (first-fit, best-fit, buddy, slab) determines speed, fragmentation, and memory overhead. No single strategy works well for all workloads, and the allocator must choose without knowing future allocation patterns.
