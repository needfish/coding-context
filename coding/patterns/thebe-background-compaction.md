# thebe -- background compaction

## Problem

A storage system accumulates data over time. To reclaim space and keep reads fast, it reorganises data in the background (merging fragments, removing garbage, improving locality) while concurrent reads and writes must see a consistent view.

## Input

- Source storage: segments, pages, or log files to compact
- A compaction strategy: when to compact, what to compact, how to merge

## Output

- Compacted storage: less total space, fewer fragments, faster reads
- The read/write path always sees either the old data or the new data after compaction completes

## Observable Behavior

- A background process reads source data, merges/rewrites it, and atomically replaces old data with new data
- Before atomic publication, reads see the old data; after, they see the new data
- No partial compaction results are visible to readers
- No data is lost: all source data is present in the compacted output (unless explicitly deleted as garbage)
- Compaction does not block reads or writes beyond a brief pause for atomic publication

## Minimal Code

```python
import os, json

SEGMENT_DIR = "segments/"

def compact():
    segments = sorted(os.listdir(SEGMENT_DIR))
    if len(segments) < 2:
        return
    
    merged = []
    for seg in segments:
        with open(os.path.join(SEGMENT_DIR, seg)) as f:
            merged.extend(json.load(f))
    
    merged.sort(key=lambda x: x["key"])
    
    # Write to temp file, then atomically rename
    tmp = os.path.join(SEGMENT_DIR, ".compact.tmp")
    with open(tmp, "w") as f:
        json.dump(merged, f)
    
    for seg in segments:
        os.remove(os.path.join(SEGMENT_DIR, seg))
    
    os.rename(tmp, os.path.join(SEGMENT_DIR, "segment_0.json"))
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Write amplification | The compactor rewrites the same data many times, using more I/O than the original writes | Tune the compaction trigger threshold; use tiered compaction |
| Read amplification | Without compaction, data is spread across many files, reads must scan all of them | Compact frequently enough to bound the number of files |
| Compaction stalls reads | A large compaction uses so much I/O that foreground reads are delayed | Throttle compaction to a configurable I/O budget |
| Crash during compaction | System crashes while compactor is writing new data, leaving inconsistent state | Use a write-ahead log or atomic rename; on restart, discard incomplete output and retry |
| Infinite compaction loop | New data is written as fast as the compactor can compact it | Use level-based or tiered compaction that bounds total work |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Leveled vs. tiered compaction | Leveled (merge into next level, one file per level) has lower read amplification but higher write amplification. Tiered (merge within a tier, multiple files per tier) has lower write amplification but higher read amplification. |
| Synchronous vs. asynchronous | Synchronous (block writes until compaction finishes) is simple but reduces write throughput. Asynchronous (compaction runs in background, writes continue) maintains throughput but needs careful coordination. |
| Full vs. partial | Full compaction rewrites everything, expensive but best space recovery. Partial rewrites only the most fragmented regions, cheaper but leaves some fragmentation. |

## Start With

Periodically merge all segments into one when the number of segments exceeds a threshold (as in the minimal code). Add level-based compaction when write amplification becomes a problem.

## Split Into Its Own Module When

- The storage system accumulates more than 10 segments or files over time.
- Read performance degrades as data accumulates.
- Write amplification from compaction becomes a concern.
- Compaction needs to be throttled based on system load.

## Signs This Is Present

- A background task periodically merges or compacts data files.
- A database or storage engine supports compaction.
- Log files or segments accumulate and need periodic cleanup.
- Read performance degrades over time until a maintenance operation runs.

## Common Mistakes

- Compacting during peak load (schedule during low-load periods or throttle).
- Not testing crash recovery during compaction.
- Using a merge strategy that does not bound the number of segments.
- Blocking reads during compaction (use atomic pointer swap or versioning).
- Not accounting for disk space: compaction may need free space equal to the data being compacted.
