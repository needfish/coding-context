# pandora -- simple workpieces

## Problem

A user edits a document (text, structured data, image) through a sequence of operations: insert, delete, modify. Each operation changes the document state. The user can undo (reverse the last operation) and redo (reapply the last undone operation). The document must be valid after every operation.

## Input

- An edit operation (a function that transforms the document)
- For undo/redo: a history of past operations or document states

## Output

- A new document state after the operation is applied
- For undo: the previous document state before the last operation
- For redo: the state after the last undone operation

## Observable Behavior

- After each operation, the document is in a valid state
- Undo reverses the most recent operation and restores the document to its previous state
- Redo reapplies the most recently undone operation
- Any operation that has been applied can be undone
- After an undo, making a new edit clears the redo stack (the undone operations cannot be redone)
- The undo stack has a limited size (oldest states are discarded when the limit is reached)

## Minimal Code

```python
class Document:
    def __init__(self):
        self.content = ""
        self.history = [""]  # snapshots
        self.history_index = 0

    def apply(self, operation):
        self.content = operation(self.content)
        self.history = self.history[:self.history_index + 1]
        self.history.append(self.content)
        self.history_index += 1

    def undo(self):
        if self.history_index > 0:
            self.history_index -= 1
            self.content = self.history[self.history_index]

    def redo(self):
        if self.history_index < len(self.history) - 1:
            self.history_index += 1
            self.content = self.history[self.history_index]
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Undo breaks invariants | Undoing an operation produces a document state that was never possible through forward operations | Snapshot the document after each operation and restore from snapshot on undo |
| Redo stack cleared unexpectedly | User undoes, then makes a new edit, and the undone operations are gone | This is correct behaviour; branching history is complex |
| Memory exhaustion on large documents | Saving a full snapshot after every keystroke consumes too much memory | Store operations (deltas) instead of snapshots, or snapshot periodically |
| Concurrent edit conflicts | Two users edit the same document simultaneously, producing conflicting states | Use Operational Transformation or CRDT |
| Cursor/selection lost on undo | Undo restores document state but cursor jumps to the start | Save cursor/selection position alongside each snapshot |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Full-snapshot vs. delta-based undo | Full-snapshot (save entire document before each edit) is simple but memory-intensive. Delta-based (store inverse operation) is memory-efficient but needs the inverse of every operation. |
| Fine-grained vs. coarse-grained operations | Fine-grained (insert character) gives precise undo but creates many history entries. Coarse-grained (paste paragraph) creates fewer entries but makes undo less granular. |
| Stateless vs. stateful document | Stateless (derived from operation list) is easy to undo/redo but slow to read. Stateful (current state in memory) is fast to read but needs careful undo handling. |

## Start With

Use full-snapshot undo with an array of document states (as in the minimal code above). Switch to delta-based undo when profiling shows memory is a problem.

## Split Into Its Own Module When

- The document is more complex than a single string (tree, graph, rich text).
- Undo/redo needs to be differential (delta-based) for memory efficiency.
- The document is shared between multiple users.
- Document validation is needed after each edit.

## Signs This Is Present

- A document or file is opened for editing.
- Undo/redo functionality is present.
- A sequence of edits is tracked in a history list.
- Operations are applied to a central data structure.

## Common Mistakes

- Allowing undo to restore a state that was never explicitly created by the user.
- Not bounding the undo stack size (memory exhaustion on long editing sessions).
- Making operations that cannot be reversed (side effects like network calls).
- Storing the full document without compression for large documents.
- Not handling the initial empty state (undo should do nothing when the document is unchanged).
