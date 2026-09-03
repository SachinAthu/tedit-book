# 31.3 Strategy 1 - Snapshots

The simplest way to implement undo is to save a copy of the document before each change.

Instead of recording how to reverse an operation, we record the **state of the document itself**.

Consider:

```text
Initial document

Hello
```

Before an edit, the editor creates a snapshot:

```text
Snapshot
─────────────
Hello
```

The user then inserts ` world`:

```text
Current document

Hello world
```

If the user presses Undo, the editor simply restores the snapshot:

```text
Snapshot
─────────────
Hello

        ↓ restore

Current document
─────────────
Hello
```

This makes snapshots an attractive first approach because the undo operation is conceptually straightforward.

## The Basic Idea

A snapshot is a saved representation of the document at a particular point in time.

Conceptually:

```text
Document State
      │
      │ copy
      ▼
+-------------+
|  Snapshot   |
+-------------+
```

When an edit is about to occur:

```text
Current Document
       │
       ├──────────────► Snapshot
       │
       ▼
Perform Edit
```

The editor therefore maintains two concepts:

```text
Current State
     │
     │
     ▼
+-----------+
| Document  |
+-----------+

History
     │
     ▼
+-----------+
| Snapshot  |
+-----------+
```

The snapshot is not the active Document. It is a historical copy that can be restored later.

## A Sequence of Snapshots

Suppose the user performs several edits:

```text
Initial:
Hello

Edit 1:
Hello!

Edit 2:
Hello! World

Edit 3:
Hello! World.
```

A snapshot-based history could contain:

```text
Snapshot 0
─────────────
Hello

Snapshot 1
─────────────
Hello!

Snapshot 2
─────────────
Hello! World
```

The current document is:

```text
Hello! World.
```

If the user presses Undo:

```text
Current
Hello! World.
     │
     │ restore Snapshot 2
     ▼
Hello! World
```

Another Undo restores Snapshot 1:

```text
Hello! World
     │
     │ restore Snapshot 1
     ▼
Hello!
```

Another restores Snapshot 0:

```text
Hello!
     │
     │ restore Snapshot 0
     ▼
Hello
```

The editor is effectively moving backward through previously recorded states.

## Why Snapshots Are Attractive

Snapshots have a major advantage:

**They are easy to understand.**

The editor does not need to determine how to reverse every possible operation.

It does not need to remember:

```text
Which character was deleted?

Where was it deleted?

Which line was split?

Where was the split?

What text was inserted?
```

Instead, it simply remembers:

```text
What did the document look like before the edit?
```

That makes the restoration process conceptually simple:

```text
Undo
  │
  ▼
Take previous snapshot
  │
  ▼
Replace current document state
```

For an educational editor, this simplicity is valuable because it allows us to understand the behavior of undo before introducing the additional machinery required by operation-based history.

## Snapshots and the Document Model

Our Document is represented as a collection of lines.

For example:

```text
Document
├── Line 0: "Hello"
├── Line 1: "world"
└── Line 2: "editor"
```

A snapshot must therefore preserve the relevant Document contents:

```text
Snapshot
├── Line 0: "Hello"
├── Line 1: "world"
└── Line 2: "editor"
```

The snapshot needs to be independent from the active Document.

This is important.

Suppose the active Document and the snapshot both referenced the same line storage:

```text
Document ─────┐
              ├──► "Hello"
Snapshot ─────┘
```

If the Document modifies that memory, the supposed historical snapshot would change as well.

The result would no longer be a snapshot.

Therefore:

> **A snapshot must preserve its own copy of the state it represents.**

Conceptually:

```text
Document
   │
   │ deep copy
   ▼
Snapshot

Document ───► its own data
Snapshot ───► its own data
```

The two representations can then evolve independently.

## Snapshot Granularity

There is an important design question:

> **What exactly should a snapshot contain?**

The simplest answer is the entire Document.

```text
Snapshot
└── Complete Document
```

But an editor has more state than text alone.

For example:

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

We should therefore distinguish between a **document snapshot** and a **complete editor-state snapshot**.

A document snapshot preserves:

```text
Document
```

A broader editor-state snapshot might preserve:

```text
Document
Cursor
Preferred Column
```

The latter can provide more accurate restoration of the editing experience, but it also increases what must be copied and managed.

For the baseline concept, the important requirement is that the snapshot preserve whatever state the undo operation is expected to restore. The exact boundary should be chosen deliberately rather than copying the entire Editor State indiscriminately.

## The Cost of a Snapshot

The simplicity of snapshots comes with a price.

Suppose the document contains:

```text
1 MB of text
```

A complete snapshot requires approximately another:

```text
1 MB
```

of document storage, ignoring the additional metadata and allocation overhead of the Document representation.

If the editor stores ten such snapshots:

```text
Current Document
      +
Snapshot 1
      +
Snapshot 2
      +
...
      +
Snapshot 10
```

the memory requirement can grow substantially.

This is the fundamental trade-off:

```text
Snapshots
   │
   ├── Simple undo
   │
   └── More memory
```

The larger the document and the more history the editor retains, the more significant this cost becomes.

## Copying Also Takes Time

Snapshots do not only consume memory.

Creating a complete snapshot requires copying the document.

If the document contains `n` characters, creating a complete copy requires work proportional to the amount of data being copied.

Conceptually:

```text
Document
[====================]
         │
         │ copy
         ▼
Snapshot
[====================]
```

This means a large document can make snapshot creation expensive even when the actual edit is tiny.

For example:

```text
Document size: 100 MB

User action:
insert one character
```

The actual edit may affect only a single position, but a full snapshot potentially requires copying the entire 100 MB document.

This illustrates an important engineering distinction:

> **The cost of an edit and the cost of recording history do not have to be the same.**

A snapshot-based design makes them closely related because recording history means copying the state.

## Snapshot History as a Stack

Snapshots naturally fit the stack model introduced later in the chapter.

Imagine:

```text
          Top
           │
           ▼
     +-----------+
     | Snapshot 3|
     +-----------+
     | Snapshot 2|
     +-----------+
     | Snapshot 1|
     +-----------+
```

The most recent snapshot is the first one needed by Undo.

Therefore:

```text
Undo
  │
  ▼
Remove top snapshot
  │
  ▼
Restore it
```

This follows the **Last In, First Out (LIFO)** behavior of a stack.

The stack does not need to understand the meaning of the document. It simply preserves the chronological order of historical states.

## Snapshot-Based Redo

Snapshots can also support Redo.

Suppose the history is:

```text
Snapshot 1
Snapshot 2
Snapshot 3
```

and the current document corresponds to the state after Snapshot 3.

After Undo, the editor needs to preserve the state it just left so that it can move forward again.

Conceptually:

```text
Undo Stack              Redo Stack

Snapshot 1
Snapshot 2              Current State
```

Another Undo moves another state into the redo history.

A Redo operation reverses that movement:

```text
Redo
  │
  ▼
Take state from Redo history
  │
  ▼
Restore it
```

The exact mechanics of maintaining the two histories will be developed later. For now, the important idea is that **snapshots can provide the historical states required in both directions**.

## A Practical Example

Consider this sequence:

```text
Initial:
Hello

Edit:
Hello world

Edit:
Hello world!

Undo:
Hello world

Redo:
Hello world!
```

A snapshot-oriented view might look like:

```text
History

S0 → Hello
S1 → Hello world
S2 → Hello world!
```

Current position:

```text
S0 → S1 → S2
             ▲
           Current
```

After Undo:

```text
S0 → S1 → S2
        ▲
      Current
```

After Redo:

```text
S0 → S1 → S2
             ▲
           Current
```

The user experiences undo and redo as movement through editing history, while the implementation can achieve that behavior by restoring saved states.

## The Strength of the Snapshot Approach

The greatest strength of snapshots is **conceptual simplicity**.

A snapshot-based system does not need every editing operation to define an inverse.

Whether the user performs:

```text
Insert character
Delete character
Split line
Merge lines
```

the snapshot mechanism can treat them all in essentially the same way:

```text
Save state
   ↓
Perform operation
   ↓
State changes
```

Undo then follows one general rule:

```text
Restore previous state
```

This makes snapshots particularly useful when first designing an undo system.

## The Fundamental Limitation

The weakness is equally clear:

> **Snapshots preserve complete states even when only a tiny part of the document changed.**

Consider:

```text
10 MB document
```

and a one-character edit:

```text
A → B
```

An operation-based history might only need information about that character.

A snapshot-based history may preserve the entire 10 MB document.

Therefore:

```text
                 Memory Efficiency

Snapshots       ████████████████
Operations      ███
```

The exact difference depends on the workload and representation, but the underlying trade-off remains.

Snapshots exchange memory efficiency for implementation simplicity.

## When Snapshots Make Sense

Snapshots are particularly attractive when:

- documents are small
- the undo history is short
- implementation simplicity is important
- predictable restoration is more valuable than memory efficiency
- the editor is educational or experimental

For a small terminal editor, these properties make snapshots a reasonable strategy to study.

However, as documents and histories become larger, the cost becomes harder to justify.

That is why professional systems often use more sophisticated approaches.

## Engineering Insight

A useful engineering lesson emerges from this strategy:

> **The simplest correct design is often the best place to begin, even when it is not the final optimized design.**

Snapshots give us a clean model of what undo and redo need to accomplish:

```text
Edit
  ↓
Preserve state
  ↓
Change document
  ↓
Navigate backward or forward through history
```

Once that model is understood, we can evaluate more efficient designs without losing sight of the underlying problem.

The next step is to examine an important question that snapshot-based undo raises:

**How often should the editor create snapshots, and what exactly should each snapshot contain?**

## Summary

Snapshots make undo conceptually simple by preserving a complete copy of the document state before each change. Undo restores the previous snapshot, and the same preserved states can support redo, but the approach exchanges memory efficiency and copying cost for implementation simplicity.

Now that the snapshot strategy is clear, the next section examines the design choices it raises.

Continue to **Section 31.4, Snapshot Granularity**.
