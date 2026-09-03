# 31.1 Why Undo and Redo Exist

A text editor changes the document every time the user inserts or deletes text. Those changes are usually easy to perform, but they create an important problem:

**What happens when the user makes a mistake?**

Consider a simple document:

```text
Hello world
```

The user accidentally deletes `world`:

```text
Hello
```

The editor now has the new document state, but the previous state is no longer directly available.

Undo exists to solve this problem.

```text
Before edit
    │
    │ delete " world"
    ▼
After edit
    │
    │ Undo
    ▼
Before edit
```

Redo solves the opposite problem. If the user undoes an operation and then decides they actually wanted it, the editor should be able to apply that change again.

```text
Original
   │
   │ Edit
   ▼
Modified
   │
   │ Undo
   ▼
Original
   │
   │ Redo
   ▼
Modified
```

The important observation is that **undo and redo are not ordinary editing operations**. They are mechanisms for navigating through the history of editing operations.

## A Simple Editing History

Suppose the user starts with:

```text
Hello
```

They then perform three edits:

```text
1. Insert "!"
2. Insert " World"
3. Delete "H"
```

The document passes through several states:

```text
Hello
  │
  │ insert "!"
  ▼
Hello!
  │
  │ insert " World"
  ▼
Hello! World
  │
  │ delete "H"
  ▼
ello! World
```

The editor could think of this as a sequence of changes:

```text
Edit 1 → Edit 2 → Edit 3
```

If the user presses Undo, the most recent change is reversed:

```text
ello! World
      │
      │ Undo
      ▼
Hello! World
```

Another Undo reverses the previous change:

```text
Hello! World
      │
      │ Undo
      ▼
Hello!
```

And another:

```text
Hello!
   │
   │ Undo
   ▼
Hello
```

Undo therefore works backward through editing history.

## Redo Moves Forward

After undoing an operation, the editor has a second problem.

Suppose the user has reached:

```text
Hello!
```

after undoing the insertion of `" World"`.

If they press Redo, the editor should restore that change:

```text
Hello!
   │
   │ Redo
   ▼
Hello! World
```

The history can therefore be viewed as two directions:

```text
Earlier states                         Later states

   Hello ──────► Hello! ──────► Hello! World ──────► ello! World
                    ▲                 ▲
                    │                 │
                  Undo              Redo
```

Undo moves toward earlier editing states.

Redo moves toward later editing states.

## Undo Requires Memory

This leads to the central idea of the chapter:

> **An editor cannot undo a change unless it has retained enough information to reconstruct what existed before that change.**

Consider this operation:

```text
Hello
```

becoming:

```text
Helo
```

because the second `l` was deleted.

To undo the deletion, the editor needs to know at least:

```text
What was deleted?
Where was it deleted?
```

For example:

```text
Deleted character: 'l'
Position: 2
```

The editor can then reconstruct:

```text
Helo
 ↓
Hello
```

The same principle applies to more complicated operations.

For an insertion:

```text
Position
Inserted text
```

For a deletion:

```text
Position
Deleted text
```

For a line split:

```text
Line
Column
```

The exact information depends on the operation.

## Undo Is More Than "Go Back"

It is tempting to think of undo as simply restoring an older copy of the document.

That is one possible implementation, but it is not the only one.

There are two broad approaches:

```text
             Undo
              │
       ┌──────┴──────┐
       ▼             ▼
   Snapshots     Operations
       │             │
Store states     Store changes
```

A snapshot-based system remembers previous document states.

An operation-based system remembers what changed and enough information to reverse those changes.

Both approaches can produce the same user-visible behavior:

```text
Edit
  ↓
Undo
  ↓
Previous document state
```

but their memory usage, implementation complexity, and performance characteristics are different.

We will examine snapshots and operation-based history separately later in this chapter.

## Undo Is About Changes, Not Every Event

Our editor processes many kinds of commands.

For example:

```text
Move cursor
Insert character
Delete character
Scroll
Save
Quit
```

Not all of these should normally become undoable history entries.

If the user moves the cursor:

```text
Hello|
```

to:

```text
|Hello
```

they generally do not expect:

```text
Undo
```

to move the cursor back.

Undo is primarily concerned with **document-changing operations**.

A useful distinction is:

```text
Event
  │
  ▼
Command
  │
  ├── changes document ──► history
  │
  └── does not change document ──► no history entry
```

This distinction builds directly on the Command Engine introduced earlier. The Command Engine is responsible for executing commands; the undo system adds the ability to retain the information necessary to reverse commands that modify the document.

## The Core Problem

We can now state the problem more precisely.

When an editing command changes the Document:

```text
Document
   │
   │ command
   ▼
New Document
```

the editor must retain enough information to answer:

> **What change occurred, and what information is required to reverse it?**

Undo then becomes:

```text
Current Document
       │
       │ reverse recorded change
       ▼
Previous Document State
```

Redo adds another requirement:

```text
Undo
  │
  ▼
Earlier state
  │
  │ redo
  ▼
Later state
```

This means a complete undo/redo system needs some form of **history management**.

In the following sections, we will examine the different ways that history can be represented, beginning with the simplest approach: **snapshots of the document state**.

## Summary

Undo and redo solve the problem of mistaken edits by navigating through editing history rather than by performing ordinary editing operations. The central requirement is that the editor must retain enough information to reconstruct what existed before each change, which leads to two broad strategies: preserving complete states (snapshots) or preserving the changes themselves (operations).

Now that the problem is defined, the next section clarifies precisely what an undo operation means and what history must contain.

Continue to **Section 31.2, What Does an Undo Operation Mean?**.
