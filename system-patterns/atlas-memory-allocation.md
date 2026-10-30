# atlas -- memory allocation

## Problem

A program needs to reserve blocks of memory of different sizes for different uses, then release them back when done. The strategy for finding and recycling free blocks varies by workload. Different strategies give different trade-offs between speed, memory waste, and fragmentation.

## Input

- Allocation request: number of bytes needed
- Free request: a pointer to previously allocated memory

## Output

- Allocation: a pointer to a block of at least the requested size, or an error (out of memory)
- Free: nothing (the block is returned to the pool)

## Observable Behavior

- Each allocation returns a block that does not overlap with any other live allocation
- Freeing a valid pointer makes that memory available for future allocations
- Freeing an invalid pointer (double-free, never-allocated) causes undefined behaviour
- After many allocations and frees, the pool may have enough total free space but no contiguous block large enough for a request (fragmentation)

## Minimal Code

Use the language's built-in allocator. No custom allocator needed until profiling shows it is a bottleneck.

```c
// C
void *ptr = malloc(1024);
free(ptr);
```

```rust
// Rust
let mut v = Vec::with_capacity(1024);
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Fragmentation | Total free memory is sufficient but allocation fails because no single contiguous block is large enough | Switch allocation strategy (buddy allocator, slab allocator, compacting GC) |
| Memory leak | Allocated blocks are never freed; program memory use grows over time until OOM | Track allocations; use RAII or garbage collection |
| Double-free | Program crashes with heap corruption | Use tools (Valgrind, ASan) to detect double-frees |
| Use-after-free | Program crashes or reads garbage from a freed block | Set pointer to NULL after free; use Rust borrow checker |
| False sharing | Two threads write to different data on the same cache line; program runs slower than expected | Align data to cache line boundaries |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Speed vs. fragmentation | Simple free list (first-fit) is fast but fragments. Buddy allocator is slower but fragments less. |
| General vs. specialised | General-purpose allocator handles any size but is slower for specific patterns. Pool allocator is fast for fixed-size objects but cannot handle variable sizes. |
| Thread-safe vs. single-threaded | Locking every allocation is safe but slow. Thread-local caches are fast but risk imbalance. |

## Start With

Use the language's default allocator (malloc/free in C, default allocator in Rust, built-in in Python/JS). Add a pool allocator when profiling shows many allocations of the same size.

## Split Into Its Own Module When

- Many objects of the same size need to be allocated (pool allocator helps).
- Allocation latency must be bounded (real-time requirements).
- The default allocator causes unacceptable fragmentation.
- Allocation statistics are needed for debugging or profiling.

## Signs This Is Present

- A custom memory allocator is being considered.
- The application uses many small, short-lived allocations.
- Memory fragmentation causes OOM errors despite enough total free memory.
- Deterministic allocation latency is required.

## Common Mistakes

- Writing a custom allocator before profiling shows the default one is a problem.
- Using a general-purpose allocator for fixed-size objects.
- Ignoring alignment requirements of the platform.
- Not handling OOM gracefully.
