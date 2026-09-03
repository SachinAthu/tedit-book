# 31.9 What Counts as an Undoable Action?

Not every command executed by the editor should become an entry in the undo history.

Undo is concerned primarily with **changes to the document**. Commands that only change the user's view or interact with the surrounding system generally do not need to be undoable.

This distinction keeps the history meaningful and prevents unrelated editor activity from appearing in the user's editing history.

## Document-Changing Actions

Typical undoable actions include operations that modify the contents or structure of the document:

```text
Insert text
Delete text
Split a line
Merge lines
Replace text
```

For example:

```text
Before:
hello|

Insert " world"

After:
hello world|
```

The insertion should create an undo record because it changed the document.

Likewise, deleting text creates a history entry containing enough information to restore what was removed.

## Non-Undoable Actions

Many editor commands do not modify the document at all.

Examples include:

```text
Move cursor
Scroll viewport
Search
Show a message
Resize the terminal
Refresh the screen
```

These actions normally should not create undo records.

For example, if the user does this:

```text
Type "hello"
Move cursor
Scroll down
Move cursor
```

the undo history should generally contain only the text insertion:

```text
Undo History:
    Insert "hello"
```

The cursor movements and scrolling should not appear as separate undo steps.

Otherwise, pressing Undo could produce surprising behavior such as moving the cursor backward through previous locations before actually undoing the user's edit.

## The Important Boundary

The key question is not:

> "Was a command executed?"

Instead, ask:

> **"Did this command create a document change that the user would reasonably expect Undo to reverse?"**

This gives the Command Engine a useful conceptual boundary:

```text
                    Command
                       │
             ┌─────────┴─────────┐
             │                   │
       Changes Document     Does Not Change
             │                 Document
             ▼                   ▼
       Undoable Action       No History
             │
             ▼
       Undo History
```

This distinction is particularly important because the editor contains many commands that operate on different parts of the Editor State.

## Document State vs. Editor State

The Editor State contains more than the document itself.

Conceptually:

```text
Editor State
├── Document
├── Cursor
├── Viewport
├── Mode
├── Status information
└── Message information
```

A command may modify one of these components without modifying the document.

For example:

```text
Cursor movement
    → Cursor changes
    → Document unchanged
    → Usually not undoable

Text insertion
    → Document changes
    → Cursor may also change
    → Undoable
```

This is why undo history should not simply record every modification to the Editor State.

The history represents **editing history**, not a complete timeline of every internal state transition.

## Commands That Change Multiple Things

A document-changing command may also change other editor state.

Suppose the user inserts a character:

```text
Before:
Document: "abc"
Cursor:   (0, 3)

Insert "X"

After:
Document: "abcX"
Cursor:   (0, 4)
```

The operation primarily changes the document, but the cursor also moves.

Undo therefore raises an important design question:

```text
Undo insertion
       │
       ├── Restore document?
       └── Restore cursor position?
```

The answer depends on the editor's intended user experience and history model. This will be examined separately when we discuss **cursor position and editor state** later in the chapter.

The important point here is that **the reason an action enters history is its semantic role as an editing operation**, not simply the number of internal fields it changes.

## Compound Actions

Some user actions may result in several low-level document modifications.

For example, a command that inserts a newline might conceptually require:

```text
Split line
    ↓
Create new line
    ↓
Move text
    ↓
Update cursor
```

From the user's perspective, however, this is one editing action:

```text
Press Enter
    ↓
One Undo
```

If every internal modification became a separate history entry, the user might need several Undo operations to reverse one visible action.

Therefore, undo history should be based on **user-meaningful operations**, rather than blindly recording every low-level mutation.

This idea leads directly to history grouping, which will be discussed later.

## A Practical Classification

A useful starting classification is:

| Action          | Document changes? | Usually undoable? |
| --------------- | ----------------: | ----------------: |
| Insert text     |               Yes |               Yes |
| Delete text     |               Yes |               Yes |
| Split line      |               Yes |               Yes |
| Merge lines     |               Yes |               Yes |
| Replace text    |               Yes |               Yes |
| Move cursor     |                No |                No |
| Scroll viewport |                No |                No |
| Search          |                No |                No |
| Display message |                No |                No |
| Save document   |              No\* |                No |

\*Saving changes the filesystem and may update editor metadata such as the dirty flag, but it is not normally treated as an editing operation in the document's undo history.

The exact classification can evolve as the editor gains more features, but the principle remains stable:

> **Undo history should contain meaningful reversible editing actions, not every command or every internal state change.**

This keeps Undo aligned with what users actually mean when they ask the editor to "undo my last change."

## Summary

Not every command should become a history entry. Undo history should contain meaningful reversible editing actions—operations that change the Document—while cursor movement, scrolling, and similar non-document changes remain outside history. Compound actions should be treated as one user-meaningful operation rather than as every low-level mutation.

Now that the boundary of undoable actions is clear, the next section examines how undo interacts with cursor position and editor state.

Continue to **Section 31.10, Cursor Position and Editor State**.
