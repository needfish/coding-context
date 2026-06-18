# callisto: Filesystem Walk

## Problem

A directory tree must be traversed recursively, visiting every entry (file, directory, symlink) while respecting exclusion rules, following (or not following) symlinks, and controlling depth.

## Context

Arises whenever a system needs to enumerate filesystem contents — search tools, backup software, file managers, build systems, linters. The walk is almost never the terminal subproblem — visited entries feed filter (venus), display (mercury), transform (neptune), or command execution (io).

## Domains Involved

- **Root path** (lexical): the starting directory path(s) for the traversal.
- **Filesystem** (causal): the underlying OS filesystem that provides directory listings and entry metadata.
- **Exclusion rules** (lexical): patterns that determine which entries to skip — `.gitignore`, `.fdignore`, hidden file rules, custom ignore patterns.
- **Visitor** (causal or lexical): the consumer of walked entries. May be a callback (causal — called for each entry) or a channel (lexical — entries pushed to a buffer).

## Shared Phenomena

- Walker → Filesystem: `readdir()`, `lstat()` calls
- Filesystem → Walker: entry names, metadata (type, size, permissions, timestamps)
- Exclusion rules → Walker: glob patterns, override rules, hidden/ignore flags
- Walker → Visitor: `DirEntry` (path + metadata), traversal errors

## Invariants

- Every entry in the tree must be visited exactly once, unless excluded by rules.
- Symlinks must be followed only if explicitly configured; the default is to return the symlink entry itself, not its target.
- Exclusion rules must be evaluated deterministically — the same root and rules must produce the same set of visited entries.
- Traversal must complete in bounded time proportional to the tree size; it must not hang on cyclic directory structures.
- Errors on individual entries (permission denied, broken symlink) must not abort the entire walk.

## Known Failure Modes

- **Symlink cycle**: A symlink points to a parent directory, causing infinite recursion without cycle detection.
- **Permission denied**: An unreadable directory causes the walk to skip an entire subtree silently, or abort with errors.
- **Filesystem change during walk**: Entries are created, moved, or deleted during traversal — the walk sees an inconsistent snapshot.
- **Deep nesting**: A tree exceeds the OS or application recursion limit, causing stack overflow.
- **Ignore rule conflict**: Two exclusion patterns contradict, causing an entry to be both included and excluded.
- **Encoding mismatch**: Filesystem paths use an encoding that the caller cannot decode, causing garbled output or skipped entries.
- **Network filesystem latency**: Walking over NFS/SMB with high latency causes the walk to stall.

## Forces

- **Breadth-first vs. depth-first**: BFS finds all entries at shallow depth first (responsive display); DFS finds entries in a single branch completely before moving on (better cache locality).
- **Sequential vs. parallel**: Sequential walk is simple and predictable; parallel walk (multiple threads reading different directories) is faster but introduces ordering nondeterminism.
- **Eager vs. lazy filtering**: Applying exclusion rules during the walk (eager) reduces memory but couples walk to filter; filtering after the walk (lazy) keeps walk generic but stores more entries.
- **Inode ordering vs. name ordering**: Natural filesystem order is inode (arbitrary); most tools need name order, which requires buffering and sorting.

## Related Subproblems

- **venus/filter**: Walked entries are almost always filtered by pattern, type, size, or time.
- **mercury/display**: Walked entries are typically displayed to the user.
- **io/exec**: Walked entries may be passed to an external command.
- **mars/persist**: Walk results may be stored.
- **earth/sort**: Results may need sorting before display or processing.

## Frame Concern

**The gap between filesystem order and consumer order.** The filesystem returns entries in inode order (arbitrary and unstable). Most consumers expect sorted or deterministic order. Buffering all entries for sorting works for small sets but fails for large trees. The concern is how to reconcile the filesystem's natural ordering with the consumer's expected ordering without unbounded memory.
