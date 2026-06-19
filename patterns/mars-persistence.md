# mars: Persistence

> **Pressure class:** Object variation  --  the storage backend changes while the access contract stays the same.

## Problem

Data must be stored and retrieved across system lifetimes, surviving process termination, restart, and failure.

## Context

Arises whenever state must outlive the execution context that produces or consumes it. Persistence is a supporting subproblem  --  it rarely drives the decomposition on its own. It couples to display (mercury) for read paths, validate (jupiter) for write paths, and transform (neptune) for serialisation.

## Domains Involved

- **Working set** (lexical): in-memory data that must be stored.
- **Storage medium** (causal): the external system that holds the data across lifetimes. A file system, a database, a blob store, a network service.
- **Schema** (lexical): the mapping between the working set structure and the storage medium's representation.

## Shared Phenomena

- Write path: Working set  ->  Schema  ->  Storage (data, serialised bytes)
- Read path: Storage  ->  Schema  ->  Working set (bytes, deserialised data)
- Schema  ->  Both: field definitions, type mappings, constraints

## Invariants

- A write must either complete entirely or leave the storage state as if the write never happened (atomicity).
- A read after a completed write must return the written value (durability after ack).
- Concurrent reads and writes must not produce inconsistent results (isolation).
- The working set and the stored representation must be isomorphic at the level of the application's data model.

## Known Failure Modes

- **Partial write**: A failure during write corrupts the stored data or leaves it in an inconsistent state.
- **Stale read**: A read returns old data because a concurrent write hasn't been flushed.
- **Schema drift**: The working set structure evolves but the stored data uses the old schema.
- **Resource exhaustion**: The storage medium fills, locks, or becomes unreachable.
- **Serialisation loss**: The working set contains structures (circular references, closures, unicode) that the serialiser cannot represent.
- **Impedance mismatch**: The working set model (object graph, relations, documents) does not align with the storage model.

## Forces

- **Speed vs. safety**: Synchronous writes to disk are slow; buffering and caching improve speed but risk data loss on crash.
- **Consistency vs. availability**: Strong consistency limits concurrent access; relaxed consistency accepts stale reads for higher throughput.
- **Normalisation vs. locality**: Normalised storage avoids duplication but requires joins to reconstruct the working set; denormalised storage duplicates data but makes reads faster.
- **Schema flexibility vs. integrity**: A flexible schema accepts any structure but cannot enforce constraints; a rigid schema enforces integrity but resists evolution.

## Related Subproblems

- **mercury/display**: Display reads from persisted data.
- **jupiter/validation**: Data must be validated before writing.
- **neptune/transformation**: Data may need serialisation/deserialisation.
- **saturn/routing**: Queries may need routing to the correct storage backend.
- **uranus/authentication**: Access to storage may require authentication.

### ==== Practical Use ====

#### When this pattern matters
Any project where state must survive process termination  --  configuration, user data, game saves, caches, logs.

#### Start with (minimal form)
A single JSON/TOML/YAML file read on startup and written on shutdown. Python: `json.load` / `json.dump`. Rust: `serde_json::from_reader` / `serde_json::to_writer`. No database, no repository abstraction.

#### Split into its own module when (observable triggers)
- You need incremental saves (not flush-all-on-exit)  --  users lose work if the process crashes between saves.
- Two or more concurrent readers/writers access the same data file.
- You need transactional semantics (partial writes must roll back on failure).
- The storage format needs versioned migration (v1  ->  v2 schema changes).

#### Signs you're in this pattern
- Your program loses state when it restarts  --  configuration, user data, or progress is gone.
- You're writing data to a file, but the format keeps changing.
- Two parts of the system read/write the same data and sometimes see inconsistent values.
- A crash during save corrupts the stored file, requiring manual recovery.

#### Real project examples
- **Flowstate** (`state.go`, `driver.go`): Execution state is persisted atomically  --  step completion + state write is a single transaction. Multiple backends (mem, Postgres, BadgerDB) under the same interface.
- **DaemonDB** (`wal.go`, `bufferpool.go`): WAL + buffer pool provide transactional durability. Write-ahead logging serialises changes before they reach the B-tree.
- **tiny-lsm-go** (`mem_table.go`, `wal.go`): LSM-tree persists in-memory writes to a WAL before acknowledging  --  crash recovery replays the WAL.
- **miniaudio** (device config persistence): Audio device configuration (sample rate, buffer size) stored in a simple JSON file, read once on startup.

#### Common over-engineering traps
- Adding a database (SQLite, PostgreSQL) when a JSON file suffices.
- Building a repository abstraction layer on day 1 before you know what queries you need.
- Implementing atomic write (write-to-temp + rename) before confirming partial writes are a real problem.

## Frame Concern

**The durability guarantee boundary.** The system promises that data survives beyond the current execution, but the storage medium has its own failure modes (crash, corruption, disconnection). Any solution must define where the guarantee ends  --  what counts as "success" and what the caller must do when the guarantee cannot be met.
