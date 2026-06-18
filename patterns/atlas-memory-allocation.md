# atlas: Memory Allocation

> **Pressure class:** Object variation — the allocation strategy changes while the allocate/free contract stays the same.

## Problem

A finite pool of memory blocks must be carved up for variable-sized allocation requests and recycled when freed, while minimising fragmentation, bounding overhead, and supporting concurrent access patterns.

## Context

Arises in any system that manages dynamic memory: general-purpose allocators (malloc), RTOS heap implementations, slab allocators for kernel objects, embedded pool allocators, game engine frame allocators, and garbage-collected heaps. The allocator mediates between the operating system's page-level memory interface and the application's byte-level allocation requests. Different workloads (many small allocations, few large ones, high concurrency, real-time bounds) demand different strategies.

## Domains Involved

- **Memory pool** (lexical): the contiguous or segmented region of addressable memory managed by the allocator.
- **Chunks** (lexical): contiguous sub-regions within the pool that can be allocated or free.
- **Allocation request** (causal): a call to `malloc(n)` or equivalent, specifying a size and optionally alignment and lifetime hints.
- **Free request** (causal): a call to `free(p)` returning a previously allocated chunk to the pool.
- **Allocator** (causal): the data structures and logic that satisfy allocation requests by finding a suitable chunk and splitting or coalescing as needed.
- **Free list** (lexical): a data structure indexing available chunks by size, address, or both.
- **Arena** (lexical): a per-thread or per-CPU sub-allocator that reduces lock contention.
- **Cache** (lexical): a thread-local pool of pre-split chunks for frequently used sizes (tcache in jemalloc).

## Shared Phenomena

- Application → Allocator: allocation request with size
- Allocator → Memory pool: split, coalesce, commit, decommit operations
- Allocator → Free list: insert freed chunk, look up suitable chunk by size
- Allocator → OS: `mmap`/`sbrk`/`VirtualAlloc` when pool is exhausted, `munmap`/`madvise` when surplus
- Allocator → Application: pointer to allocated region (or null on failure)
- Application → Allocator: free request with pointer
- Arena → Allocator: load-balance allocations across arenas by thread ID or round-robin

## Invariants

- Every allocated chunk must be uniquely identifiable by its starting address (no overlapping allocations).
- Freeing an invalid pointer or double-freeing must be detected or prevented (guard pages, quarantine, magic numbers).
- The allocator must not leak memory between allocation contexts — freeing in one thread must correctly return to the pool available to other threads.
- Coalescing adjacent free chunks must never produce a chunk larger than the maximum allocatable size.
- The allocator must maintain an upper bound on internal fragmentation (worst-case waste per allocation).
- The total memory managed by the allocator must equal the sum of allocated + free + overhead (metadata).
- All metadata structures (free list pointers, size headers, arena control blocks) must themselves be allocated without circular dependency.

## Known Failure Modes

- **External fragmentation**: Free memory is fragmented into many small chunks, and a large allocation request cannot be satisfied despite sufficient total free space.
- **Internal fragmentation**: An allocation request rounds up to a larger size class, wasting the rounded-up excess within the allocated chunk.
- **Blowup**: The allocator's metadata overhead grows beyond acceptable limits (e.g., one metadata word per allocation in a system with millions of small allocations).
- **False cache-line sharing**: Allocators that place unrelated objects on the same cache line cause performance degradation on multi-core systems.
- **Lock contention**: A single global free list serialises all allocation/free calls under high concurrency, becoming a bottleneck.
- **Memory leak**: Allocated chunks are never freed, causing monotonic pool growth until exhaustion.
- **Use-after-free**: A freed chunk is reallocated while a dangling pointer still references it, causing corruption.
- **Heap overflow**: A write past the end of an allocated chunk corrupts adjacent metadata or data.
- **OOM**: The pool is exhausted and the OS refuses to supply more memory; the allocator must surface an error without crashing the process in an inconsistent state.

## Forces

- **Speed vs. fragmentation**: A simple free-list allocator (first-fit) is fast but fragments badly; a segregated-fit allocator (size-class bins) minimises fragmentation but incurs overhead for bin management.
- **Concurrency vs. simplicity**: Per-thread arenas eliminate lock contention but increase total memory footprint due to per-arena fragmentation; a single global heap is space-efficient but serialises.
- **Generality vs. specialisation**: A general-purpose allocator must handle any request size; specialised allocators (object pools, frame allocators) can be faster and more space-efficient for specific workloads.
- **Bounded time vs. throughput**: Real-time allocators (TLSF) bound worst-case allocation time to O(1) but sacrifice throughput for the general case; general allocators optimise for throughput at the cost of unpredictable worst-case times.
- **Return-to-OS vs. caching**: Returning freed memory to the OS reduces process footprint but incurs syscall overhead on reallocation; caching freed memory avoids syscalls but increases footprint.

## Related Subproblems

- **pan/resource-scheduling**: Memory allocation requests compete for the same allocator locks as application threads, coupling allocation latency with scheduling.
- **mars/persist**: Persistent memory allocators (for flash, NVRAM) must be power-fail safe and wear-level aware.
- **jupiter/validate**: Bounds checking, canary values, and address sanitizers validate allocator correctness.
- **neptune/transform**: Allocators transform raw page-granular memory from the OS into byte-granular chunks for the application.
- **hyperion/network-listener**: Network packet buffers are allocated and freed per-packet, making allocator throughput a primary factor in network performance.

### ==== Practical Use ====

#### When this pattern matters
Any project that manages dynamic memory — most non-trivial software outside embedded systems with no heap.

#### Start with (minimal form)
Use the language's built-in allocator. Rust: `Box`, `Vec`, `HashMap`. Python: automatic allocation. C: `malloc`/`free`. No custom allocator, no memory pool.

#### Split into its own module when (observable triggers)
- Allocation latency is causing measurable performance problems (benchmarks show allocator as bottleneck).
- Many small, short-lived allocations cause fragmentation in the default allocator.
- Real-time constraints require bounded allocation time (O(1) pool allocator).
- You need to track allocation statistics or detect leaks programmatically.

#### Common over-engineering traps
- Writing a custom slab allocator before measuring that the default allocator causes fragmentation.
- Building per-thread arenas before confirming that lock contention on the global allocator is measurable.
- Implementing memory pooling for a type that is allocated infrequently.

## Frame Concern

**The gap between allocation request sizes and the underlying page granularity of virtual memory.** The application requests arbitrary byte counts, but the OS provides memory only in page-sized (4 KiB or larger) units. The allocator must bridge this gap by splitting pages into smaller chunks and managing the resulting inventory, but its splitting and coalescing strategies determine how well it can satisfy future requests. When the inventory cannot satisfy a request (fragmentation), the allocator must request more pages, potentially exacerbating the gap further.
