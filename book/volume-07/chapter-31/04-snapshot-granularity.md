# 31.4 Snapshot Granularity

Once we decide to use snapshots, another design question appears:

> **How much state should each snapshot preserve?**

A snapshot does not have to mean "copy everything in the editor." We must decide what constitutes the historical state that Undo and Redo are expected to restore.

This decision is called **snapshot granularity**.

## Whole-Document Snapshots

The simplest approach is to snapshot the entire Document.

Suppose the current document is:

```text
Hello
World
Editor
```

A snapshot contains the complete document:

```text
Snapshot
├── "Hello"
├── "World"
└── "Editor"
```

After an edit:

```text
Hello
Beautiful World
Editor
```

the new state can be represented by another complete snapshot:

```text
Snapshot
├── "Hello"
├── "Beautiful World"
└── "Editor"
```

The history therefore contains complete versions of the Document.

```text
S0
│
├── Hello
├── World
└── Editor

S1
│
├── Hello
├── Beautiful World
└── Editor
```

This approach is easy to reason about because every snapshot is self-contained.

## Why Whole-Document Snapshots Are Simple

Suppose the user performs a complicated operation involving several lines.

With an operation-based system, the history record might need to preserve:

```text
Position
Deleted text
Inserted text
Affected lines
Cursor position
Other operation-specific information
```

With a whole-document snapshot, none of that needs to be reconstructed.

The snapshot simply says:

```text
This is what the Document looked like.
```

Undo can restore that complete state.

This makes the mechanism independent of the particular editing operation.

```text
Insert
Delete
Split
Merge
      │
      ▼
  Snapshot
      │
      ▼
Restore
```

That uniformity is the primary reason snapshots are attractive.

## The Cost of Whole-Document Snapshots

The same simplicity creates a significant cost.

Consider a document containing 1 MB of text.

If the user makes ten edits and the editor keeps ten complete snapshots, the history may contain roughly:

```text
Current Document   ≈ 1 MB
Snapshot 1         ≈ 1 MB
Snapshot 2         ≈ 1 MB
Snapshot 3         ≈ 1 MB
...
Snapshot 10        ≈ 1 MB
```

The document data alone could therefore require roughly 11 MB, before considering the internal allocations and metadata used by the Document representation.

The important point is not the exact number.

It is the relationship:

```text
History memory
     ↑
     │
Document size × number of snapshots
```

As either document size or history depth increases, memory consumption increases.

## Snapshot Per Keystroke

The most straightforward policy might be:

> Create a snapshot before every editing operation.

For example, typing:

```text
Hello
```

could produce:

```text
Snapshot 0
""

Snapshot 1
"H"

Snapshot 2
"He"

Snapshot 3
"Hel"

Snapshot 4
"Hell"

Snapshot 5
"Hello"
```

This provides extremely fine-grained undo.

The user can undo one character at a time:

```text
Hello
Hell
Hel
He
H
""
```

But it also demonstrates the memory problem very clearly.

A tiny sequence of edits can create many complete document copies.

## Coarser Snapshot Granularity

The opposite approach is to create snapshots less frequently.

For example, the editor could group several related edits into one historical state:

```text
User types:

H e l l o

       ↓

One history entry

"Hello"
```

Undo would then remove the complete editing action rather than necessarily reversing every individual keystroke.

This reduces the number of snapshots:

```text
Instead of:

S0 → S1 → S2 → S3 → S4 → S5

we might have:

S0 → S1
```

The trade-off is obvious:

```text
Fine-grained history
        │
        ├── More precise undo
        └── More memory

Coarse-grained history
        │
        ├── Less precise undo
        └── Less memory
```

Snapshot granularity is therefore both a memory decision and a user-experience decision.

## What Belongs in a Snapshot?

There is another dimension to granularity.

The editor contains more than the Document.

Our Editor State includes concepts such as:

```text
Editor State
├── Document
├── Cursor
├── Viewport
├── Current File
├── Dirty Flag
├── Screen Size
├── Status Message
└── Configuration
```

Should all of these be copied?

Generally, **no**.

Many of these values describe the current editing environment rather than the historical document state.

For example, the terminal's screen size should not become part of undo history.

Likewise, a status message such as:

```text
File saved successfully
```

has no meaningful relationship to an undoable document edit.

A snapshot should therefore have a deliberate boundary.

## Document State vs Editor State

It is useful to distinguish these concepts:

```text
Document State
├── Text contents
└── Information necessary to represent the document

Editor State
├── Document
├── Cursor
├── Viewport
├── Current File
├── Dirty Flag
└── Other session state
```

Undo primarily concerns the **Document State**.

However, some editing operations may also require related editor information to provide a natural user experience.

The cursor is the clearest example.

Consider:

```text
Before:
Hello|
```

After insertion:

```text
Hello World|
```

After Undo:

```text
Hello|
```

Restoring the document without considering the cursor could leave the user in an unexpected position.

Therefore, a practical undo design may preserve selected pieces of editor state alongside the document snapshot.

The important principle is:

> **Do not snapshot the entire Editor State simply because it is convenient. Define which state actually belongs to editing history.**

## Why Viewport Usually Does Not Belong

Consider a user scrolling through a document without changing it.

The viewport might change from:

```text
viewport_row = 0
```

to:

```text
viewport_row = 20
```

But the Document has not changed.

There is normally no reason for:

```text
Undo
```

to restore the previous viewport.

The viewport represents **where the user is looking**, not a modification to the document.

This distinction keeps history focused:

```text
Document change
       │
       ▼
Undo history

Viewport movement
       │
       ▼
No document history
```

This follows the same boundary established earlier between document-changing commands and navigation commands.

## Snapshot Granularity and User Actions

There is a deeper question:

> What should the user perceive as one undoable action?

Suppose the user types:

```text
The quick brown fox
```

Internally, the editor may receive many individual character events:

```text
T
h
e
space
q
u
i
c
k
...
```

But the user may think of the entire typing sequence as one continuous action.

This creates a difference between:

```text
Low-level operations
```

and:

```text
User-level actions
```

A history system must eventually decide where one undoable unit begins and ends.

That problem becomes particularly important for operation-based history, where individual operations can be grouped into larger commands.

For snapshot-based history, the same issue appears as the question of **when to take the snapshot**.

## Snapshot Granularity Is a Policy

There is no universally correct granularity.

Different editors can make different choices depending on their goals.

A system optimized for simplicity might choose:

```text
Every edit → new snapshot
```

Another system might group related changes:

```text
User action → one snapshot
```

A more sophisticated system might use operation records and dynamic grouping.

The important engineering lesson is:

> **History granularity is a user-experience policy backed by a storage strategy.**

It should not be treated as an accidental consequence of the implementation.

## Snapshot Boundaries

A useful way to think about snapshots is as boundaries between historical states:

```text
        Snapshot          Snapshot
           │                 │
           ▼                 ▼
State A ─────── Edit ─────── State B
```

If several low-level operations belong to one user action:

```text
State A
   │
   ├── operation 1
   ├── operation 2
   ├── operation 3
   │
   ▼
State B
```

then the history system can regard the entire sequence as one transition.

This produces a more useful undo experience without changing the underlying Document model.

## Choosing a Baseline

For our educational editor, the most important goal at this stage is not to design the most sophisticated history system possible.

We want a model that is:

- easy to understand
- deterministic
- easy to test
- consistent with the existing architecture
- small enough to implement without introducing unnecessary machinery

Whole-document snapshots provide exactly that starting point.

However, their memory cost becomes increasingly unattractive as documents and histories grow.

That limitation gives us a reason to investigate another approach rather than attempting to optimize snapshots indefinitely.

## The Trade-off

Snapshot granularity can be summarized as:

| Granularity           | Undo Precision | Memory Usage | Complexity |
| --------------------- | -------------- | -----------: | ---------: |
| Every low-level edit  | Very fine      |         High |        Low |
| Grouped edits         | Coarser        |        Lower |   Moderate |
| Large state intervals | Coarse         |       Lowest |   Moderate |

The correct choice depends on what the editor considers a meaningful editing action.

For our purposes, the important conclusion is:

> **Snapshots make undo simple by storing states, but deciding how frequently and how broadly to snapshot determines both memory consumption and the quality of the undo experience.**

That leads naturally to the next strategy.

Instead of preserving complete states, we can preserve the **operations that produced those states**. This is where the Command Pattern becomes useful.

## Summary

Snapshot granularity decides how often snapshots are taken and how much state each one preserves. Whole-document snapshots are simple and self-contained but their memory cost grows with document size and history depth, and the correct boundary is a policy choice that keeps history focused on document state rather than copying the entire Editor State.

Now that the snapshot approach and its limitations are clear, the next section introduces the alternative of recording the operations that changed the document.

Continue to **Section 31.5, The Command Pattern**.
