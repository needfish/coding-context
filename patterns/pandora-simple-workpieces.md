# pandora: Simple Workpieces

## Problem

A user interacts with a structured document (text, spreadsheet, diagram, source code) through a sequence of editing operations. The system must maintain the document's internal consistency, support undo and redo across operations, keep multiple views synchronised, and preserve the editing state across sessions.

## Context

Arises in any application that creates and modifies documents: text editors, IDEs, spreadsheet editors, image editors, CAD tools, and note-taking apps. The workpiece has internal structure (lines, cells, layers, nodes) that must remain consistent after every edit. The user expects to reverse mistakes (undo), reapply reversed operations (redo), view the same document in multiple windows, and return to the same editing state after closing and reopening the document. The system must track which edits have been saved (dirty flag) and prompt the user before discarding unsaved work.

## Domains Involved

- **Document model** (lexical): the internal representation of the workpiece — typically a sequence of lines (text), a grid of cells (spreadsheet), or a graph of nodes (diagram editor).
- **Document view** (causal): a projection of the document model onto the screen, with scroll offset, selection state, and visual annotations.
- **Edit operation** (causal): a single atomic change to the document (insert character, delete line, paste, replace).
- **Transaction** (lexical): a group of edit operations that are applied and undone together as a unit (e.g., a single "paste" may insert dozens of characters, but the entire paste is a single undo step).
- **Undo stack** (lexical): the history of committed transactions, navigable backwards.
- **Redo stack** (lexical): transactions popped from the undo stack, available for reapplication.
- **Dirty flag** (lexical): a boolean indicating whether the in-memory document differs from the persisted copy on disk.
- **Selection** (lexical): the range(s) of the document that the user has highlighted and that subsequent operations will act on.

## Shared Phenomena

- User → Editor: edit commands (type, delete, paste, cut, indent)
- Editor → Document model: atomic edit operations constrained to maintain structural invariants
- Document model → Undo stack: composed transactions pushed on each command boundary
- Undo stack → Document model: reversed operations applied on undo, original operations stored on redo
- Document model → Views: change notifications (line inserted, lines deleted, selection changed)
- Views → Screen: redrawn regions after change notification
- Editor → File system: read on open, write on save

## Invariants

- Every undo must restore the document to the exact state before the corresponding transaction was applied.
- Undo must reverse the effects of a transaction without side effects — no other state changes may leak.
- The undo stack must not grow unbounded; old transactions may be discarded under memory pressure, but only from the oldest end.
- A new edit after an undo (but no redo) must discard the redo stack — the undone state becomes the new head.
- The dirty flag must be set on any edit that modifies the document and cleared on save.
- The dirty flag must be computed correctly across undo/redo — undoing back to the saved state must clear the flag.
- All views of the same document must reflect the same document state at all times; a change in one view must invalidate all others.
- The selection must remain valid after every edit — if the selection spans a region that was deleted, it must clamp to the available range.

## Known Failure Modes

- **Stale undo pointers**: After an undo, synthesised edit operations (auto-indent, bracket completion) create new undo entries that reference the undone state, creating a corrupted history.
- **Selection drift**: Undo restores document content but the selection position is not restored, leaving the cursor in a different location than when the original edit was made.
- **Transaction coalescing mismatch**: Adjacent typing is coalesced into one transaction for undo efficiency, but the coalescing threshold is too aggressive and merges edits the user thinks of as separate.
- **Dirty flag desync**: The saved-state checkpoint shifts after undo past the save point, but the dirty flag is not recalculated, causing the user to either lose work on close or be prompted unnecessarily.
- **Multi-view desync**: A second view (minimap, split pane) is not invalidated after an edit in the primary view, showing stale content.
- **Large document undo blowup**: Storing the entire document state per undo entry consumes too much memory; storing only the delta requires correct recomposition on undo.
- **Encoding corruption**: The document stores content in one encoding (UTF-8) but the file system uses another, causing save/load to produce different byte sequences even though the logical content is the same.

## Forces

- **Undo granularity vs. memory**: Storing every keystroke as a separate undo entry gives fine-grained undo but uses memory; coalescing keystrokes into time-bounded transactions (e.g., 2-second window) reduces memory but loses per-character undo.
- **Full snapshot vs. delta**: Storing the full document state per undo entry is simple but wasteful; storing a delta (inverse operations) is memory-efficient but requires careful composition.
- **Selection preservation vs. edit correctness**: Restoring the exact pre-edit selection after undo may place the cursor in an invalid position if the surrounding content changed; clamping the selection is safer but may not match the user's expectation.
- **Document model complexity vs. edit efficiency**: A rope data structure scales well for large documents but is more complex than a simple array of lines; the array is simple but degrades on large insertions.

## Related Subproblems

- **mars/persist**: The save/load cycle bridges the in-memory workpiece to durable storage.
- **neptune/transform**: The document model transforms user intent (key presses) into structural changes to the document.
- **mercury/display**: Views project the document model onto the screen, translating internal coordinates (line/column) to screen coordinates (pixel offset).
- **saturn/route**: The command system routes user input through key bindings to specific editing commands.
- **titan/syntax-highlighting**: The highlighter is an incremental computation over the document — it must re-highlight only the changed region.
- **atlas/memory-allocation**: The document model must manage memory efficiently for large files (line count, total bytes).

## Frame Concern

**The gap between edit operations and document state.** The user thinks in terms of edits (insert a word, delete a line, paste a block), but the document is a persistent structured state that must remain internally consistent across every edit boundary. An insert operation, for example, not only adds text but shifts all subsequent line and column positions, invalidates selection ranges, marks the document as dirty, updates syntax highlighting for the affected region, and pushes a reversable transaction onto the undo stack — all of which must happen atomically. The gap is that the user's conceptual edit ("I typed the letter 'a'") is a single action, but the system's response touches nearly every subsystem at once, and any subsystem that fails to update correctly leaves the document in an inconsistent state.
