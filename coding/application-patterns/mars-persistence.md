# mars -- persistence

## Problem

Data in memory must be saved to storage (file, database, cloud object store) and loaded back later, even after the program restarts. The storage type may be a file, a database, or a remote service.

## Input

- For save: data to persist (any structured value)
- For load: a key or path identifying the stored data

## Output

- For save: success or failure (disk full, permission denied, network timeout)
- For load: the previously saved data, or a default value if none exists

## Observable Behavior

- Data that is successfully saved can be loaded back to the same value
- A failed save leaves the previously saved data unchanged
- Two saves at the same time do not corrupt each other's data
- Loading data that was never saved returns a default value or empty state

## Minimal Code

```python
import json, os

SAVE_PATH = "data.json"

def save(data):
    with open(SAVE_PATH, "w") as f:
        json.dump(data, f)

def load():
    if not os.path.exists(SAVE_PATH):
        return {}
    with open(SAVE_PATH) as f:
        return json.load(f)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Partial write | Program crashes mid-write, file contains truncated data | Write to temp file, then rename atomically |
| Concurrent write corruption | Two threads write to same file, file contains garbage | Use file locking, a WAL, or a database |
| Backward compatibility failure | Old version of program cannot read data written by new version | Add version number to data format, write migration functions |
| Data loss on power failure | OS buffers writes, power loss loses buffered data | Use fsync() after critical writes |
| Slow startup | Loading a large dataset blocks program start | Lazy-load or stream data |

## Design Trade-offs

| Choice | What changes |
|---|---|
| File vs. database | File is simple and portable but can't query data. Database adds queries, indexes, and concurrency but is another dependency. |
| Text vs. binary format | Text (JSON, YAML) is human-readable but larger and slower. Binary (protobuf, msgpack) is compact and fast but opaque. |
| Embedded vs. external storage | Embedded (SQLite) needs no server. External (PostgreSQL, S3) scales independently but needs a separate service. |
| Optimistic vs. pessimistic locking | Optimistic (version number) works for low contention. Pessimistic (mutex, database lock) limits concurrency. |

## Start With

The minimal code above using a JSON file. Add versioning and migration when the data format changes. Add a database when querying or concurrency is needed.

## Split Into Its Own Module When

- More than one data type needs persistence.
- Data size exceeds 10MB or save/load time exceeds 100ms.
- Transactions or atomic multi-record operations are needed.
- Migration between schema versions is required.
- Multiple backends (file, S3, database) must be supported.

## Signs This Is Present

- Data is written to and read from a file, database, or cloud store.
- A serialisation library (json, pickle, protobuf) is used.
- Data loading happens at startup and saving at shutdown or on change.
- Migration scripts exist for schema version upgrades.

## Common Mistakes

- Not handling the first-run case (file does not exist, database is empty).
- Assuming atomic writes on all platforms without using temp-file + rename.
- Not testing for disk-full conditions.
- Using pickle for long-term storage (version sensitivity, security).
- Storing derived data alongside source data.
- Not documenting what happens on power loss.
