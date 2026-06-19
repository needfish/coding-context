# thebe: Background Compaction

> **Pressure class:** Behavior variation  --  the compaction policy algorithm changes per storage structure and access pattern.

## Problem

Storage that accumulates incremental changes (writes, inserts, log entries) must be periodically reorganised in the background to maintain read performance, reclaim space, and bound metadata growth  --  all while concurrent foreground reads and writes continue without blocking.

## Context

Arises in any system with an append-only or log-structured write path: LSM-tree storage engines, log-structured file systems, journaling systems, message queues with retention policies, versioned blob stores, and incremental backup systems. Writes are fast (append to the head), but reads require scanning multiple on-disk structures, and deleted or overwritten data consumes space until reclaimed. A background process reorganises the accumulated structures into a more compact form, merging, discarding stale entries, and producing new consolidated structures that replace the old ones atomically. The foreground workload must not stall during this process.

## Domains Involved

- **Working set** (lexical): the collection of on-disk structures that hold the current data. In an LSM-tree: the MemTable (in-memory writes) and SST files (on-disk sorted runs) across levels.
- **Compaction worker** (causal): the background process that reads existing structures, merges them, discards stale entries, and writes new consolidated structures.
- **Level/index metadata** (lexical): the mapping from key ranges to the structures that contain them. Must be updated atomically when compaction produces new structures.
- **Foreground reader** (causal): concurrent read operations that must observe a consistent view of data during compaction.
- **Foreground writer** (causal): concurrent write operations that must not be blocked by compaction.

## Shared Phenomena

- Compaction worker  ->  Working set: reads existing structures (SST files, segments)
- Compaction worker  ->  Storage: writes new consolidated structures
- Compaction worker  ->  Level metadata: atomic swap (new structures go live, old structures are retired)
- Foreground reader  ->  Level metadata: lookup key  ->  find which structure(s) to search
- Foreground writer  ->  Working set: append to active write buffer (MemTable, WAL segment)
- Compaction worker  ->  Foreground operations: resource contention (disk I/O bandwidth, CPU)

## Invariants

- Every key-value pair in the old structures must appear in the new structures unless it has been deleted or superseded by a newer write.
- The atomic swap must be visible to foreground readers instantaneously  --  they must never see a partial compaction result.
- Foreground reads must succeed during compaction  --  they may read from old or new structures depending on timing, but must always return a consistent value for any key.
- Compaction must not exceed the configured I/O bandwidth budget (it must throttle to avoid starving foreground operations).
- Compaction must eventually terminate for a given set of input structures  --  no infinite loops or unbounded growth.
- Compaction must not be triggered so aggressively that it competes with foreground writes (write amplification bound).

## Known Failure Modes

- **Write stall**: The active write buffer fills up while compaction is busy and cannot accept new writes, causing foreground writes to block.
- **Read amplification growth**: Compaction falls behind the write rate, so the number of on-disk structures grows, and each read must scan more files.
- **Space amplification**: Compaction does not reclaim deleted/overwritten entries fast enough, causing storage usage to grow beyond acceptable limits.
- **Compaction storm**: A large compaction job suddenly consumes all I/O bandwidth, causing foreground latency spikes.
- **Torn compaction**: The system crashes during compaction, and on recovery the old structures are still present but partially compacted data is orphaned.
- **Level overflow**: Data is compacted to the deepest level, but that level fills up and compaction has nowhere to push data, causing a compaction loop.
- **Key range overlap**: Compaction produces structures whose key ranges overlap with existing structures, violating the level invariant and causing inconsistent reads.

## Forces

- **Write amplification vs. space amplification**: Frequent compaction keeps space usage low but causes more writes to storage; infrequent compaction saves write bandwidth but wastes space.
- **Latency vs. throughput**: Throttling compaction to avoid latency spikes reduces compaction throughput, potentially causing write stalls; running compaction at full speed maximises throughput but causes foreground latency spikes.
- **Level-based vs. size-tiered compaction**: Level-based compaction (LevelDB/RocksDB style) limits read amplification at the cost of higher write amplification; size-tiered compaction (Cassandra style) has lower write amplification but higher read amplification.
- **Full vs. partial compaction**: Compacting all levels together maximises space efficiency but is expensive; compacting only adjacent levels is cheaper but leaves more space amplification.

## Related Subproblems

- **pan/resource-scheduling**: Compaction I/O must be scheduled alongside foreground operations with appropriate priority and throttling.
- **mars/persistence**: The storage structures being compacted are persistence concerns (SST files, WAL segments, log files).
- **atlas/memory-allocation**: Compaction may allocate significant memory for merge buffers and intermediate results.
- **venus/filtering**: Bloom filters on compacted structures must be rebuilt after compaction.
- **earth/sorting**: Merging multiple sorted runs during compaction is a merge-sort operation.
- **phoebe/resource-lifecycle**: Compaction workers and the SST files they produce follow an acquire/process/release lifecycle.

### ==== Practical Use ====

#### When this pattern matters
Any system with log-structured or append-only storage  --  LSM-tree databases, message queues, time-series databases, blob stores with versioning, incremental backup systems.

#### Start with (minimal form)
No compaction at all. Reclaim space by rewriting the entire storage from scratch during maintenance windows. For trivial sizes, compact synchronously when active storage exceeds a threshold.

```go
// Simplest form: trigger sync compaction when memtable is full
func (db *DB) compact() {
    all := db.readAllSSTs()
    merged := mergeAndDiscardStale(all)
    db.writeSST(merged)
    db.deleteOldSSTs()
}
```

#### Split into its own module when (observable triggers)
- The storage holds more data than can be rewritten in a maintenance window.
- Foreground reads become slow because too many on-disk structures must be scanned.
- The write rate consistently exceeds the compaction rate (write stalls occur).
- You need to control compaction I/O bandwidth independently from foreground reads/writes.
- Multiple levels or tiers of compaction are needed for the write/read amplification trade-off.

#### Signs you're in this pattern
- Your storage system accumulates redundant data over time (old versions, deleted entries, fragmented pages).
- Reads become slower over time because data is scattered across many files/pages.
- A background process reorganises storage while the system continues serving reads/writes.
- You need bounded read/write amplification and must compact to maintain it.

#### Real project examples
- **tiny-lsm-go** (`compaction.go`): Level-based compaction  --  multiple SST files at level N are merged into a single file at level N+1. Single-threaded compaction with size-tiered triggering.
- **DaemonDB** (B-tree page compaction): Pages are compacted during checkpoint  --  merges underfull pages and reclaims free space. Triggered by free-page ratio crossing a threshold.
- **lite3** (B-tree defrag): Serialisation format uses B-trees with opportunistic compaction on write  --  free space is reclaimed during insert/update paths.

#### Common over-engineering traps
- Implementing level-based compaction (multiple levels with fan-in ratios) before confirming that full-merge compaction is too slow.
- Adding concurrent compaction (multiple compaction workers) before measuring that single-threaded compaction is the bottleneck.
- Building a sophisticated I/O bandwidth scheduler before confirming that compaction causes measurable foreground latency.
- Supporting dynamic throttling before static rate-limiting is shown to be insufficient.

## Frame Concern

**Consistency during concurrent reorganisation.** Compaction reads the current state, produces a new state, and atomically swaps it in  --  but foreground reads and writes operate on the same data concurrently. The concern is how to make the compaction result visible to foreground operations at a consistent point without blocking either the compaction process or the foreground operations  --  a concurrent data structure problem disguised as a storage problem.
