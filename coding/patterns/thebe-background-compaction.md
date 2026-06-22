# ------------------------------------------------------------------------------
#  thebe-background-compaction.md
#  Behavior variation -- the compaction policy algorithm changes per storage structure and access pattern.
# ------------------------------------------------------------------------------

# - Problem -

Your storage accumulates incremental changes over time -- writes, inserts, log entries. You need to periodically reorganise it in the background to keep reads fast, reclaim space, and limit metadata growth -- all while foreground reads and writes continue without blocking.

# - When You'll See This -

Any system with an append-only or log-structured write path. LSM-tree storage engines, log-structured file systems, journaling systems, message queues with retention, versioned blob stores, and incremental backup systems. Writes are fast (append to the head), but reads may need to scan multiple structures, and deleted data consumes space until reclaimed.

# - How It Works -

- **Working set**: the collection of on-disk structures holding current data. In an LSM-tree: the MemTable (in-memory writes) and SST files (on-disk sorted runs) across levels.
- **Compaction policy**: the rules for when to compact, what to compact, and what the output should look like (size ratio, tiered, leveled, or time-based).
- **Compaction job**: a single unit of work -- reading input structures, merging/discarding entries, writing new output structures.
- **Foreground workload**: concurrent reads and writes that must not be blocked by compaction.
- **Space amplification**: the ratio of on-disk data to logical data -- compaction reduces this by discarding stale entries.
- **Read amplification**: the number of structures a read must consult -- compaction reduces this by merging structures.

# - What Must Always Be True -

- Compaction doesn't block foreground reads or writes beyond a bounded pause.
- After compaction, the data is logically equivalent to what existed before (same queries, same results).
- Space amplification stays within a configured bound.
- Compaction makes progress even under sustained write load.
- Compaction can be interrupted (by shutdown) and resumed later without data loss.

# - What Goes Wrong -

- **Write stall**: the write-ahead log grows faster than compaction can keep up, forcing writes to block.
- **Compaction storm**: multiple compactions trigger simultaneously, consuming all I/O bandwidth and starving foreground operations.
- **Read amplification grows unbounded**: too many unmerged structures force every read to consult many files.
- **Space amplification grows unbounded**: deleted or overwritten data is never fully reclaimed.
- **Compaction falling behind**: on a busy server, compaction can't keep up with the write rate, and the situation worsens over time.
- **Compaction of in-use data**: compacting a structure that's currently being read produces inconsistent results.

# - Design Trade-Offs -

- **Leveled vs. tiered compaction**: leveled compaction merges data into levels with size limits (good read performance, higher write cost); tiered compaction merges within a level (good write performance, more read amplification).
- **Aggressive vs. lazy compaction**: aggressive compaction keeps space/read amplification low but uses more I/O; lazy compaction saves I/O but allows amplification to grow.
- **Synchronous vs. asynchronous**: synchronous compaction blocks writes until done (simple, predictable); asynchronous compaction runs in the background (complex, but no blocking).

# - Related Problems -

- **mars/persistence**: compaction is a storage engine concern.
- **pan/resource-scheduling**: compaction needs I/O scheduling to avoid starving foreground operations.
- **phoebe/resource-lifecycle**: compaction manages the lifecycle of on-disk structures.

# - Getting Started -

Don't compact. Append-only files with periodic full rebuilds. If the data grows too large, rewrite the entire file from scratch. No background process, no incremental merging, no level management.

# - When to Split Into Its Own Module -

- The write workload exceeds the capacity of periodic full rebuilds.
- Read amplification causes noticeable latency in production.
- You need concurrent foreground access during maintenance.
- You need configurable compaction policies for different workloads.

# - Signs You're in This Pattern -

- Your storage uses append-only or log-structured files.
- Reads need to check multiple locations because data is spread across files.
- Write throughput drops over time as more data accumulates.
- You have a background task that merges or reorganises stored data.

# - Common Mistakes -

- Implementing compaction before measuring that read amplification is actually a problem.
- Letting compaction compete with foreground I/O without rate limiting.
- Not testing compaction under sustained concurrent write load.
- Compacting data that could be deleted instead (expired TTL entries).

# - The Core Challenge -

Compaction is a background task that trades I/O for read performance and space efficiency. The challenge is doing enough compaction to keep reads fast without consuming so much I/O that foreground operations stall. The right balance depends on the workload, and the workload changes over time.
