# 31.6 Operation History

With the Command Pattern established, we can now define how editing operations are actually retained as history.

An **operation history** is an ordered collection of undoable commands that represents the document's editing past.

For example, if the user performs:

```text
Insert "Hello"
Insert " world"
Delete "world"
```

the history can be represented as:

```text
History
┌─────────────────────────┐
│ Insert "Hello"          │
│ Insert " world"         │
│ Delete "world"          │
└─────────────────────────┘
              ▲
          most recent
```

The most recent operation is the first candidate for Undo.

## History as a Sequence of Operations

Each history entry describes one meaningful editing action.

A conceptual record might contain:

```text
Operation
├── type
├── position
├── affected data
└── additional information
```

For example:

```text
INSERT_TEXT
    position: (3, 8)
    text: "hello"
```

or:

```text
DELETE_TEXT
    position: (3, 8)
    text: "hello"
```

The history therefore does not need to contain complete document states. It contains the information necessary to understand and reverse the changes that produced the current state.

## Ordering Matters

Operation history is inherently ordered.

Suppose the document begins as:

```text
abc
```

The user performs:

```text
1. Insert "X" at position 1
2. Insert "Y" at position 2
```

The resulting document depends on both operations and their order.

Undo must therefore reverse operations in the opposite order:

```text
Execute:

A → B → C

Undo:

C → B → A
```

This is why a stack is a natural representation for the undo history.

The most recent operation is always the first operation that should be reversed.

## The Undo Stack

The undo history can be represented as:

```text
┌───────────────┐
│ Operation C   │ ← next undo
├───────────────┤
│ Operation B   │
├───────────────┤
│ Operation A   │
└───────────────┘
```

When `Operation C` is undone, it is removed from the undo stack and becomes available for redo.

```text
Before:

Undo: [A, B, C]
Redo: []

After undo:

Undo: [A, B]
Redo: [C]
```

The stack therefore represents **what can currently be undone**, rather than merely storing an archival list of everything that has ever happened.

## History Entries Must Be Self-Contained

An operation should retain the information required to reverse itself later.

This is particularly important for destructive operations.

For example, consider:

```text
Before:
hello world

Delete "world"
```

The operation history must retain `"world"` somewhere:

```text
DELETE_TEXT
    position: 6
    text: "world"
```

Without the deleted text, Undo would know where the deletion occurred but would not know what content to restore.

The same principle applies to structural operations.

For example, undoing a line merge may require enough information to reconstruct the original line boundary and affected content.

> **History must preserve information that the current document can no longer provide.**

## History Is Separate from the Document

The operation history should not become part of the Document itself.

Conceptually:

```text
Editor State
├── Document
│   └── Current text
│
└── History
    ├── Undo operations
    └── Redo operations
```

The Document represents the **current content**.

History represents the **operations that produced or can restore that content**.

Keeping these responsibilities separate makes the architecture easier to reason about. The Document does not need to know whether a change was caused by a normal command, an Undo, or a Redo.

## Executing History Operations

Undo and redo should use the same underlying document operations as normal editing whenever practical.

Conceptually:

```text
Normal edit
    │
    ▼
Execute operation
    │
    └── record history


Undo
    │
    ▼
Execute inverse operation
    │
    └── move history entry


Redo
    │
    ▼
Execute original operation
    │
    └── move history entry
```

However, Undo itself must not accidentally create a new ordinary undo entry.

Otherwise:

```text
Undo C
   ↓
inverse(C)
   ↓
record inverse(C) as new history
```

would cause the history to grow instead of moving backward.

The history system therefore needs to distinguish between **normal command execution** and **history-driven execution**.

## A Useful Mental Model

The operation history can be viewed as a record of transitions:

```text
State A
   │
   │ Operation A
   ▼
State B
   │
   │ Operation B
   ▼
State C
   │
   │ Operation C
   ▼
State D
```

Undo walks backward:

```text
State D
   │
   │ inverse(C)
   ▼
State C
   │
   │ inverse(B)
   ▼
State B
```

Redo walks forward again:

```text
State B
   │
   │ B
   ▼
State C
   │
   │ C
   ▼
State D
```

This model connects the Command Pattern with the two-stack state machine from the previous section.

The **command record describes the transition**, while the **undo and redo stacks determine which transitions can currently be traversed**.

## Operation History vs. Snapshots

The two strategies can now be compared more precisely:

| Aspect                          | Snapshot History  | Operation History |
| ------------------------------- | ----------------- | ----------------- |
| History entry                   | Complete state    | Operation record  |
| Memory per small edit           | Potentially large | Usually small     |
| Undo mechanism                  | Restore state     | Apply inverse     |
| Required information            | Entire state      | Reversal data     |
| Implementation complexity       | Lower             | Higher            |
| Dependency on command semantics | Low               | High              |

Neither approach changes the fundamental user-facing behavior:

```text
Edit → Undo → Redo
```

They differ in **how the history needed to support that behavior is represented**.

For the editor architecture, operation history provides a more scalable foundation because it records meaningful changes rather than repeatedly copying the entire document.

The next step is to examine the **redo stack** in more detail, including how operations move between undo and redo history and why new edits invalidate redo history.

## Summary

Operation history is an ordered collection of undoable commands representing the document's editing past. Order matters because operations must be reversed in the opposite order from which they were executed, which is why a stack is the natural representation, and each history entry must be self-contained so it can reverse itself later.

Now that the undo stack is established, the next section examines the redo stack and how operations move between the two histories.

Continue to **Section 31.7, The Redo Stack**.
