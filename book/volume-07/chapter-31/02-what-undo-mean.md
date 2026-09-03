# 31.2 What Does an Undo Operation Mean?

Before deciding how to store undo history, we need to define what **undo** actually means.

At first glance, the idea seems simple:

> Undo should restore the document to what it was before the last edit.

But an editor does not necessarily need to store the entire previous document to achieve this. It can instead remember **what changed** and use that information to reverse the change.

## From One State to Another

Consider the document:

```text
Hello
```

The user inserts `!` at the end:

```text
Hello!
```

Conceptually, we have:

```text
State A
Hello
   │
   │ Insert '!'
   ▼
State B
Hello!
```

Undo must perform the reverse transition:

```text
State B
Hello!
   │
   │ Reverse insertion
   ▼
State A
Hello
```

The important point is that undo is not inherently a command to "load the previous document."

It is a command to **reverse a previous change**.

## The Inverse Operation

We can describe the relationship using the idea of an **inverse operation**.

If an operation transforms:

```text
A → B
```

then its inverse should transform:

```text
B → A
```

For example:

| Original operation | Inverse operation         |
| ------------------ | ------------------------- |
| Insert `A`         | Delete `A`                |
| Delete `A`         | Insert `A`                |
| Split a line       | Merge the resulting lines |
| Merge two lines    | Restore the line boundary |

The relationship can be visualized as:

```text
        Operation
A ─────────────────► B
        │
        │ inverse
        ▼
A ◄───────────────── B
```

This is the basic conceptual foundation of operation-based undo.

However, there is an important complication.

**The inverse operation may require information that is no longer present in the Document.**

## Why the Inverse Needs Information

Suppose the document contains:

```text
Hello world
```

The user deletes:

```text
world
```

The resulting document is:

```text
Hello
```

What operation would undo the deletion?

We need to insert `world` again.

But where?

```text
Hello|
```

We need at least:

```text
Deleted text: "world"
Position: after "Hello"
```

Without the deleted text, the editor cannot reconstruct the previous document.

The Document itself no longer contains that information.

Therefore, an undo record might conceptually contain:

```text
Undo Record
├── Operation: DELETE
├── Position: ...
└── Deleted Text: "world"
```

The record contains the information needed to perform the inverse operation.

## Insertion Is Different

Now consider an insertion:

```text
Hello
```

becoming:

```text
Hello world
```

To undo it, we need to remove the inserted text.

In this case, the inserted text is already present in the current Document.

The undo record may therefore need less information:

```text
Undo Record
├── Operation: INSERT
├── Position: ...
└── Length: 6
```

The editor can interpret this as:

```text
Delete 6 characters at this position
```

The exact representation can vary, but the principle remains the same:

> An undo record must preserve whatever information the inverse operation cannot recover from the current document.

## Line Operations Need More Context

Our editor works with a document represented as lines, so editing can also change line boundaries.

Consider:

```text
Hello world
```

The user presses Enter between the two words:

```text
Hello
world
```

This is a **line split**.

Undo must reconstruct:

```text
Hello world
```

Therefore, the editor needs to know where the split occurred.

Conceptually:

```text
Split
├── line
└── column
```

Undo can then merge the two resulting lines at that boundary.

The reverse relationship is:

```text
Split line
     │
     ▼
+---------+---------+
|  Hello  |  world  |
+---------+---------+
     │
     │ Undo
     ▼
+-------------------+
|    Hello world    |
+-------------------+
```

Again, the important part is not the particular data structure.

The important part is understanding **what information was lost by the original operation** and therefore needs to be retained for undo.

## Undo Records Describe Changes

We can now introduce a useful abstraction:

```text
Undo Record
```

An undo record represents a document-changing operation together with the information required to reverse it.

For example:

```text
Insert Character

Operation: INSERT
Position: 5
Character: 'A'
```

or:

```text
Delete Text

Operation: DELETE
Position: 10
Text: "hello"
```

or:

```text
Split Line

Operation: SPLIT_LINE
Row: 4
Column: 8
```

These records are not necessarily the final structures we will implement. They are a conceptual model for understanding what history must contain.

## Undo Is Not the Same as a Document Snapshot

This distinction is important.

A **snapshot** says:

> Here is what the document looked like.

An **operation record** says:

> Here is what changed.

For example:

```text
Snapshot
────────────────────
Hello world
```

versus:

```text
Operation
────────────────────
INSERT "world" at position 6
```

The first stores a state.

The second stores a transition.

Both can support undo, but they make different trade-offs.

```text
             Undo History
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
    Snapshots           Operations
        │                   │
   Store states        Store changes
```

We will examine snapshots first because they provide the simplest possible implementation model. Operation-based history becomes more important when we discuss the Command Pattern and operation stacks.

## Undo Must Preserve Semantics

There is another subtle point.

An undo operation should reverse the **meaningful editing action**, not merely manipulate bytes until the document happens to look correct.

For example, if the user inserts a character:

```text
A
```

then undo should conceptually mean:

```text
Remove the character that was inserted.
```

It should not depend on unrelated information such as the current cursor position if that position has changed since the insertion.

This is why an undo record should contain enough information to identify the original operation independently of subsequent navigation.

The history therefore describes **past editing actions**, not merely the current state of the editor.

## Undo and the Cursor

Undo can also affect more than document contents.

Suppose:

```text
Before:
Hello|
```

The user inserts `" world"`:

```text
Hello world|
```

After undo, the document should become:

```text
Hello|
```

The cursor has returned to its previous logical position as well.

This raises an important design question:

> Should an undo operation restore only the Document, or should it also restore relevant editor state?

For our editor, the answer depends on the specific operation and the desired editing behavior. Cursor semantics themselves were established earlier in the Cursor Engine chapter; here, the important observation is simply that **document history and cursor state can be related**.

We will return to this when discussing the integration between undo/redo and Editor State.

## The Fundamental Model

We can now describe an undoable operation as a transition with enough information to reverse it:

```text
                  Execute
                     │
                     ▼
             +---------------+
             | Document      |
             | State A       |
             +---------------+
                     │
                     │ operation
                     ▼
             +---------------+
             | Document      |
             | State B       |
             +---------------+
                     ▲
                     │
                     │ inverse
                     │
             +---------------+
             | Undo Record   |
             +---------------+
```

The history record bridges the two states.

This gives us the central rule for the rest of the chapter:

> **Undo history must preserve enough information to reconstruct the state that existed before an undoable operation.**

There are several ways to accomplish this. The simplest is to preserve complete states as **snapshots**. The more compact approach is to preserve individual operations and their inverse information.

The next section starts with the simplest strategy: **snapshot-based undo**.

## Summary

Undo is best understood as a command to reverse a previous change rather than to load a stored copy of the document. An undo record must preserve whatever information the inverse operation cannot recover from the current Document, and it should capture the semantic editing action rather than merely restoring matching bytes.

Snapshots and operation records are the two ways of carrying this information. Now that the conceptual model is established, the next section examines the simplest strategy: snapshot-based undo.

Continue to **Section 31.3, Strategy 1 - Snapshots**.
