# mars: Persist

## Problem

Data must be stored and retrieved across system lifetimes, surviving process termination, restart, and failure.

## Context

Arises whenever state must outlive the execution context that produces or consumes it. Persistence is a supporting subproblem — it rarely drives the decomposition on its own. It couples to display (mercury) for read paths, validate (jupiter) for write paths, and transform (neptune) for serialisation.

## Domains Involved

- **Working set** (lexical): in-memory data that must be stored.
- **Storage medium** (causal): the external system that holds the data across lifetimes. A file system, a database, a blob store, a network service.
- **Schema** (lexical): the mapping between the working set structure and the storage medium's representation.

## Shared Phenomena

- Write path: Working set → Schema → Storage (data, serialised bytes)
- Read path: Storage → Schema → Working set (bytes, deserialised data)
- Schema → Both: field definitions, type mappings, constraints

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
- **jupiter/validate**: Data must be validated before writing.
- **neptune/transform**: Data may need serialisation/deserialisation.
- **saturn/route**: Queries may need routing to the correct storage backend.
- **uranus/authenticate**: Access to storage may require authentication.

## Frame Concern

**The durability guarantee boundary.** The system promises that data survives beyond the current execution, but the storage medium has its own failure modes (crash, corruption, disconnection). Any solution must define where the guarantee ends — what counts as "success" and what the caller must do when the guarantee cannot be met.
