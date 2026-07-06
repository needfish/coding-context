# callisto -- filesystem walk

## Problem

A directory tree must be traversed: visit every file and directory reachable from a root path. The order of traversal (depth-first, breadth-first), filtering (skip hidden files, follow symlinks), and handling of errors (permission denied, broken symlink) vary per use case.

## Input

- A root directory path (string)
- Options: follow symlinks (yes/no), max depth, skip patterns

## Output

- A sequence of paths (one per visited file/directory), yielded one at a time
- Or error events for inaccessible paths

## Observable Behavior

- Every file and directory reachable from the root is visited exactly once (unless limited by max depth or skip patterns)
- The order of visitation is predictable (depth-first by default)
- Symlink loops cause infinite recursion if not detected
- Permission errors on one directory do not prevent visiting other directories

## Minimal Code

```python
import os

def walk(root):
    for dirpath, dirnames, filenames in os.walk(root):
        for f in filenames:
            yield os.path.join(dirpath, f)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Symlink loop | Traversal never ends, runs until memory exhausted | Keep a set of visited inodes; skip symlinks pointing to already-visited paths |
| Permission denied | Traversal crashes on a directory the program cannot read | Catch permission errors and skip the directory, continue with siblings |
| Very deep nesting | Recursion hits the stack limit | Use an explicit stack (iterative traversal) instead of recursion |
| File renamed during traversal | Same file visited twice or skipped entirely | Snapshot the directory listing before traversal, or use filesystem watches |
| Broken symlink | Symlink target does not exist, os.path operations fail | Check if symlink target exists before following; skip broken ones |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Depth-first vs. breadth-first | Depth-first uses less memory (stack of current path only). Breadth-first uses more memory (queue of all siblings) but finds shallow files first. |
| Recursive vs. iterative | Recursive is simpler but limited by stack depth. Iterative (explicit stack) handles any depth. |
| Sequential vs. parallel | Sequential is simple and predictable. Parallel (thread pool) is faster on SSDs or network filesystems but needs synchronisation. |

## Start With

Use os.walk (Python) or std::fs::read_dir (Rust) for sequential depth-first traversal. Add symlink detection and error handling when failures appear.

## Split Into Its Own Module When

- Traversal options (max depth, follow symlinks, skip patterns) accumulate.
- Parallel traversal is needed for performance.
- Traversal must support multiple roots or include/exclude filters.
- Error handling (permission, broken symlinks) needs to be configurable.

## Signs This Is Present

- os.walk, find, or recursive directory listing is used.
- Files are processed recursively from a root directory.
- A directory tree is scanned for matching files.

## Common Mistakes

- Not handling permission errors (crash on a directory the program cannot read).
- Not detecting symlink loops (infinite recursion).
- Building the full list of files in memory before processing.
- Mixing traversal logic with file processing logic.
- Not testing with empty directories, permission-denied directories, and symlink chains.
