# ------------------------------------------------------------------------------
#  mars-persistence.md
#  Object variation -- the storage backend changes while the access contract stays the same.
# ------------------------------------------------------------------------------

# - Problem -

Your program has state that needs to survive a restart. You need to save data to disk (or a database, or a cloud store) and read it back later.

# - When You'll See This -

Any project where state must outlive the current run -- configuration files, user data, game saves, caches, logs. Persistence usually supports other subproblems: display reads from it, validation checks data before writing to it.

# - How It Works -

- **Working set**: the in-memory data structures you want to save.
- **Storage medium**: where the data lives between runs -- a file, a database, a blob store.
- **Schema**: the mapping between your in-memory structures and the storage format.

# - What Must Always Be True -

- A write either completes fully or leaves the storage unchanged (atomicity).
- A read after a completed write returns the value that was written (durability).
- Concurrent reads and writes don't produce garbage (isolation).
- What you save and what you load back are the same at the application level.

# - What Goes Wrong -

- **Partial write**: a crash during save corrupts your data or leaves it half-written.
- **Stale read**: reading returns old data because a write hasn't been flushed yet.
- **Schema drift**: your code changes but the saved data still uses the old format.
- **Disk full**: the storage medium fills up or becomes unreachable.
- **Serialisation fails**: your data has circular references or other structures the serialiser can't handle.
- **Impedance mismatch**: your object model doesn't map cleanly to the storage model (relational vs. document vs. key-value).

# - Design Trade-Offs -

- **Speed vs. safety**: writing straight to disk is slow; buffering is fast but risks losing data on crash.
- **Consistency vs. availability**: strong consistency limits concurrent access; relaxed consistency accepts stale reads for higher throughput.
- **Normalised vs. denormalised**: normalised avoids duplication but needs joins; denormalised is faster to read but duplicates data.
- **Flexible vs. rigid schema**: flexible schemas accept anything but can't enforce constraints; rigid schemas enforce rules but are harder to change.

# - Related Problems -

- **mercury/display**: display reads from persisted data.
- **jupiter/validation**: data gets checked before writing.
- **neptune/transformation**: data often needs serialisation/deserialisation.
- **saturn/routing**: queries might need routing to the right storage backend.
- **janus/reduction**: stored data often gets read and summarised.
- **uranus/authentication**: accessing storage might need auth.

# - Getting Started -

Read and write a single JSON file on startup and shutdown. Python: `json.load` / `json.dump`. Rust: `serde_json::from_reader` / `serde_json::to_writer`. No database, no repository pattern.

# - When to Split Into Its Own Module -

- You need incremental saves (not flush-all-on-exit) -- users lose work if the process crashes between saves.
- Multiple readers or writers access the same data file.
- You need transactional rollback on failure.
- The storage format needs versioned migration (v1 -> v2 schema changes).

# - Signs You're in This Pattern -

- Your program loses state when it restarts.
- You're writing data to a file but the format keeps changing.
- Two parts of the system read and write the same data and sometimes see inconsistent values.
- A crash during save corrupts the stored file.

# - Common Mistakes -

- Adding a database (SQLite, PostgreSQL) when a JSON file works fine.
- Building a repository abstraction layer before you know what queries you need.
- Implementing atomic write before confirming partial writes are a real problem.

# - The Core Challenge -

The system promises data survives beyond the current execution, but the storage medium has its own failure modes (crash, corruption, disconnection). You need to define where the guarantee ends -- what counts as "saved" and what the caller should do when saving isn't possible.
