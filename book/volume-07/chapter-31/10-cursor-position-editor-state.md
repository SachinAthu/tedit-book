# 31.10 Cursor Position and Editor State

Undoing an edit is not always enough to restore the state the user expects.

Consider this sequence:

```text
Before:

hello|
```

The user inserts `X`:

```text
helloX|
```

After undo, the document should again contain:

```text
hello|
```

The important detail is that the cursor has also returned to the position associated with the original editing state.

This raises a design question:

> Should an undo operation restore only the Document, or should it also restore some parts of the Editor State?

The answer depends on which state represents the editing action and which state exists only to display or navigate the current view.

## Document State vs Editor State

The Document is the primary object affected by an undoable editing operation. However, editing also changes the Cursor.

For example:

```text
INSERT "abc"

Document:
    hello
        ↓
    helloabc

Cursor:
    column 5
        ↓
    column 8
```

Undoing the insertion restores both aspects of the editing experience:

```text
Document:
    helloabc
        ↓
    hello

Cursor:
    column 8
        ↓
    column 5
```

If only the Document were restored, the cursor could remain at the position where the undone text used to exist.

That would produce a technically correct Document but an unexpected editing state.

## Before and After Cursor Positions

An operation record can therefore associate cursor positions with the operation.

Conceptually:

```text
Operation
+----------------------+
| Type                 |
| Operation Data       |
| Cursor Before        |
| Cursor After         |
+----------------------+
```

For an insertion:

```text
Before:

hello|
     ^
     column 5

Operation:

INSERT "abc"

After:

helloabc|
        ^
        column 8
```

Undo can then use the stored information to restore the Document and return the Cursor to the position that existed before the operation.

Redo uses the corresponding after-position:

```text
UNDO

Document: reverse operation
Cursor:   restore Before


REDO

Document: apply operation
Cursor:   restore After
```

The exact representation of these positions is an implementation concern. The architectural principle is more important:

> If an editing operation changes the user's meaningful editing position, its history record must preserve enough information to restore that position.

## Not All Cursor Changes Are Undoable

This does **not** mean every cursor movement should become part of history.

Consider:

```text
hello|
      ↓ Left

hell|o
```

The Document has not changed.

Recording every cursor movement would make the history represent navigation rather than editing. As discussed earlier, undo history should correspond to meaningful document-changing actions.

The cursor position stored with an editing operation is therefore **metadata associated with the edit**, not a separate undoable cursor-movement operation.

This distinction keeps history meaningful:

```text
User action:
INSERT "hello"

History:
INSERT "hello"
    + cursor before
    + cursor after
```

rather than:

```text
History:
MOVE_RIGHT
MOVE_RIGHT
MOVE_RIGHT
INSERT "h"
MOVE_RIGHT
...
```

## What About the Viewport?

The Viewport is different.

Suppose an edit occurs near the bottom of the visible document:

```text
Document
    ...
    line 98
    line 99
    line 100  <--- Cursor
```

The editor may adjust the Viewport to keep the Cursor visible. Undoing the edit may cause the Cursor to move, which can require the Viewport to be adjusted again.

The Viewport therefore generally should **not** be treated as historical editing state.

Instead:

```text
Undo
  │
  ├── Restore Document
  │
  ├── Restore Cursor
  │
  └── Update Viewport
          │
          ▼
       Renderer
```

The Viewport is derived from the current editing position and screen dimensions. It describes **what portion of the Document is currently visible**, not what the Document looked like at a previous point in editing history.

This is consistent with the architecture established earlier: the Viewport belongs to Editor State, while the Renderer consumes that state to produce the current frame.

## Why Rendering State Should Not Be in History

The same principle applies even more strongly to rendering state.

Undo should not attempt to restore things such as:

- terminal screen contents
- screen refresh state
- rendered screen cells
- cursor visibility
- temporary display messages

These describe the presentation of the current state, not the editing history itself.

The Renderer should continue to derive the visual frame from the current Editor State:

```text
History
   │
   ▼
Document + Cursor
   │
   ▼
Editor State
   │
   ▼
Renderer
   │
   ▼
Current Frame
```

There is no need for undo history to preserve an old rendered frame.

## The User Experience

From the user's perspective, undo should feel as though the last editing action has been reversed.

If typing moves the cursor forward:

```text
abc|
```

then undoing that typing should return the user to the position from which the typing began:

```text
|
```

Likewise, undoing a deletion should place the cursor where the deleted text had previously been:

```text
Before deletion:

hello world|
            ^

After deleting "world":

hello |
     ^

After undo:

hello world|
            ^
```

The Document and Cursor therefore form the important historical state associated with an editing operation.

The Viewport and rendering state do not need to be historical snapshots. They should be updated to present the restored state correctly.

## Engineering Insight

Undo history should preserve **semantic editing state**, not an indiscriminate copy of Editor State.

For the baseline design, this means distinguishing:

```text
History-relevant
----------------
Document change
Cursor position associated with the change


Current-session state
---------------------
Viewport
Rendering state
Terminal state
Temporary messages
```

This distinction prevents undo from becoming a general-purpose time machine for every field in Editor State. It keeps history focused on what the user means by **“undo my edit”** while allowing the rest of the editor to recompute or update its current presentation normally.

## Summary

Undo should restore the Document and the cursor position associated with the editing action, because both form the meaningful editing state. The Viewport, rendering state, and other current-session state do not belong in history; they are derived from the restored state and recomputed by the Renderer.

Now that the historical state boundary is clear, the next section examines how undo and redo interact with the Dirty Flag.

Continue to **Section 31.11, Undo, Redo, and the Dirty Flag**.
