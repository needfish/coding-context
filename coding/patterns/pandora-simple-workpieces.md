# ------------------------------------------------------------------------------
#  pandora-simple-workpieces.md
#  Behavior variation -- the edit operation changes per user action, but the document state consistency contract stays the same.
# ------------------------------------------------------------------------------

# - Problem -

Users edit a structured document -- text, source code, a diagram, a spreadsheet. The system must maintain internal consistency, support undo and redo, keep multiple views synchronised, and preserve state across sessions.

# - When You'll See This -

Any application that creates and modifies documents. Text editors, IDEs, spreadsheet editors, image editors, CAD tools, note-taking apps. The document has internal structure (lines, cells, layers, nodes) that must stay consistent after every edit. Users expect to undo mistakes, redo reapplied operations, view the same document in multiple windows, and return to the same state after restarting.

# - How It Works -

- **Document model**: the internal representation of the workpiece -- a sequence of lines (text), a grid of cells (spreadsheet), or a graph of nodes (diagram editor).
- **Edit operation**: a single change to the document -- insert, delete, replace, transform. Each operation is reversible (has an inverse for undo).
- **Undo stack**: the history of applied operations, stored as deltas or snapshots. Popping from the stack and applying the inverse restores previous state.
- **Dirty flag**: tracks whether the document has unsaved changes since the last save.
- **View projection**: a visual representation of the document model -- multiple views can project different parts of the same model.

# - What Must Always Be True -

- Every edit leaves the document in a valid internal state (no half-applied changes).
- Undo reverses exactly one edit and restores the exact previous state.
- All views of the same document are consistent -- an edit in one view appears in all others.
- The dirty flag accurately reflects whether unsaved changes exist.
- A save persists the current state so that reopening the file restores it.

# - What Goes Wrong -

- **Undo breaks after save**: saving resets the undo stack, so users can't undo changes made before the save.
- **Inconsistent multi-view**: editing in one window doesn't update another window showing the same file.
- **Lost work on crash**: edits since the last save disappear when the process crashes.
- **Undo corruption**: applying undo after a long editing session produces a corrupted state because operations weren't correctly inverted.
- **Large undo stack**: storing full snapshots for every edit consumes too much memory.
- **Dirty flag desync**: the flag says there are unsaved changes when there aren't, or vice versa.

# - Design Trade-Offs -

- **Delta vs. snapshot undo**: storing deltas (what changed) uses less memory but is harder to implement; storing full snapshots is simple but memory-heavy.
- **Coalescing vs. individual undo**: grouping rapid keystrokes into one undo step is more useful but complicates the undo model.
- **Synchronous vs. asynchronous persistence**: saving on every edit loses no work but may be slow; saving on a timer batches writes but risks losing recent edits.

# - Related Problems -

- **mars/persistence**: documents are saved to and loaded from storage.
- **mercury/display**: the document is displayed in one or more views.
- **neptune/transformation**: document formats may need conversion.

# - Getting Started -

A text buffer (string) with insert/delete at a cursor position. Undo: store the previous string before each edit. One view. No syntax highlighting, no multi-cursor, no incremental save.

# - When to Split Into Its Own Module -

- You need multiple views of the same document (split pane, minimap, outline).
- You need coalesced undo (typing "hello" is one undo step, not five).
- The document structure is complex enough that delta-based undo matters.
- You need to preserve editing state across sessions (cursor position, undo stack).

# - Signs You're in This Pattern -

- You're building an editor or a tool that edits structured content.
- Users can undo and redo their changes.
- The document has internal structure that must stay consistent.
- Multiple UI components display the same document and need to stay in sync.

# - Common Mistakes -

- Storing the full document state for every edit when delta-based undo would use less memory.
- Not coalescing rapid edits (every keystroke is a separate undo step).
- Allowing the document to enter an invalid state during an edit.
- Forgetting to update all views when the document changes.

# - The Core Challenge -

An editor must maintain consistency at every step while providing a responsive, undoable editing experience. Each edit transforms the document from one valid state to another, and the sequence of edits must be reversible, composable, and persistent. The challenge is keeping the document model, undo stack, views, and storage synchronised through an unbounded sequence of user actions.
