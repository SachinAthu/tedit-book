# 31.5 The Command Pattern

The snapshot approach gives us a straightforward way to preserve history, but it has an important limitation: each history entry contains a copy of the document rather than a description of **what changed**.

A different approach is to record the editing operation itself. Instead of remembering the entire document before an insertion, the editor can remember something like:

> Insert `"hello"` at position `(4, 12)`.

That record contains enough information to perform the operation again or reverse it later.

## Commands as Descriptions of Changes

In our editor, a command already represents an abstract user action. For example:

```text
INSERT_TEXT
DELETE_TEXT
SPLIT_LINE
MERGE_LINE
```

The Command Pattern extends this idea by treating an editing action as a piece of data that can describe both **what happened** and **what is required to reverse it**.

Conceptually:

```text
User Action
     │
     ▼
  Command
     │
     ├── operation type
     ├── position
     ├── affected data
     └── other required information
     │
     ▼
Command Execution
     │
     ▼
Document State Changes
```

For example, an insertion might produce a history record containing:

```text
Command:
    type: INSERT_TEXT
    row: 4
    column: 12
    text: "hello"
```

To undo it, the editor does not need a copy of the entire document. It knows that `"hello"` was inserted at `(4, 12)`, so it can remove that text.

A deletion requires slightly different information:

```text
Command:
    type: DELETE_TEXT
    row: 4
    column: 12
    text: "hello"
```

The deleted text must be retained because the editor needs it to reconstruct the previous state.

## Forward and Reverse Operations

An operation record therefore has two conceptual directions:

```text
              execute
        ┌─────────────────┐
        │                 ▼
State A ───── Operation ─────► State B
        ◄───── Inverse ──────
```

For an insertion:

```text
execute:  insert "hello"
undo:     delete "hello"
```

For a deletion:

```text
execute:  delete "hello"
undo:     insert "hello"
```

The important point is that **undo does not require storing the complete previous document**. It requires storing enough information to reconstruct the previous state.

This can dramatically reduce the amount of history data for small edits.

## Commands and the Existing Command Engine

This fits naturally with the editor architecture established earlier.

The Input Engine converts input into an editor command. The Command Engine then executes that command against the Editor State.

With undo/redo, the flow becomes:

```text
Input
  │
  ▼
Input Engine
  │
  ▼
Command
  │
  ▼
Command Engine
  │
  ├── execute command
  │
  └── record history
          │
          ▼
      Undo History
```

The Command Engine therefore becomes the natural boundary for determining which document-changing commands should enter the undo history.

This does **not** mean that every command needs to become a complex object with its own methods, inheritance hierarchy, or function-pointer interface. The editor can continue using simple command values and associated data, as established by the architecture.

The Command Pattern here is primarily a **history representation**, not a reason to redesign the entire command-dispatch system.

## Why This Helps Undo/Redo

Operation-based history has several important advantages:

| Property                        | Snapshot          | Command             |
| ------------------------------- | ----------------- | ------------------- |
| Stores                          | Complete state    | Change description  |
| Small edit cost                 | Potentially large | Usually small       |
| Undo information                | Previous state    | Inverse information |
| Memory efficiency               | Lower             | Higher              |
| Complexity                      | Simple            | Higher              |
| Dependency on operation details | Low               | High                |

Consider a 10 MB document where the user inserts a single character.

A snapshot-based approach may need to preserve approximately 10 MB of document state for that edit.

An operation-based approach might only need:

```text
position + inserted character
```

The difference becomes significant when the user performs thousands of small edits.

## The Cost of Operation-Based History

The advantage comes with additional responsibility.

A command record must contain **all information required for reversal**.

If the editor deletes text but records only:

```text
DELETE_TEXT
position = (4, 12)
length = 5
```

it may know how much text was deleted, but not what the text actually was. The original content has been lost.

A correct history record must therefore preserve enough information:

```text
DELETE_TEXT
position = (4, 12)
text = "hello"
```

This leads to an important rule:

> **An undo record must preserve every piece of information that cannot be reconstructed from the current state but is required to reverse the operation.**

This is one of the central design challenges of operation-based undo.

## Command Pattern vs. Snapshot History

The two strategies should not be viewed as mutually exclusive concepts.

Snapshots preserve **states**:

```text
Snapshot A
Snapshot B
Snapshot C
```

Commands preserve **transitions**:

```text
Command 1 → Command 2 → Command 3
```

They answer different questions:

```text
Snapshots:
"What did the document look like?"

Commands:
"What changed the document?"
```

For a small educational editor, snapshots can be attractive because of their simplicity. For a more capable editor, operation-based history provides much better control over memory usage and history behavior.

The remainder of this chapter will focus primarily on the operation-based model and how those commands are organized into undo and redo history.

## Summary

The Command Pattern treats an editing action as a record that describes both what happened and what is required to reverse it. Operation-based history stores changes rather than complete states, which is memory-efficient for small edits, but every record must contain all information that cannot be reconstructed from the current document.

Now that commands can carry history information, the next section examines how those operation records are organized as an editing history.

Continue to **Section 31.6, Operation History**.
