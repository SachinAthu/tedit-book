# 16.6 Editing Commands and Core Operations

Editing commands are where the Command Engine begins to coordinate operations that change the Document.

The important distinction is that the Command Engine **does not define how text editing works**. The Document Model and Cursor Engine already define those rules. The Command Engine connects those existing operations to a user's requested command.

The relationship is:

```text
Editing Command
       │
       ▼
Command Engine
       │
       ├──────────────► Document
       │
       └──────────────► Cursor
                         │
                         ▼
                   Editor State
```

This keeps the command layer focused on coordination rather than duplicating the algorithms implemented by the core components.

## Insert Character

The `INSERT_CHARACTER` command represents a request to insert a character at the current cursor position.

Conceptually:

```text
INSERT_CHARACTER('A')
        │
        ▼
Command Engine
        │
        ├──► Document: insert 'A'
        │
        ├──► Cursor: advance position
        │
        └──► Dirty State: mark modified
```

The Document performs the actual text insertion according to its representation and editing rules.

The Cursor then reflects the new editing position.

The Command Engine coordinates these consequences so that the operation results in one consistent state transition.

It should not contain another implementation of character insertion.

## Delete Character

The `DELETE_CHARACTER` command requests removal of text relative to the current cursor position.

Conceptually:

```text
DELETE_CHARACTER
        │
        ▼
Command Engine
        │
        ├──► Document: remove text
        │
        ├──► Cursor: adjust position
        │
        └──► Dirty State: mark modified
```

The exact behavior at line boundaries belongs to the Document and Cursor rules established earlier.

For example, deleting at the beginning of a line may require the current line to be merged with the previous line. The Command Engine does not need to reproduce those rules. It coordinates the appropriate core operations.

This is an important boundary:

> **The command identifies the requested operation; the core component determines how that operation is performed correctly.**

## Split Line

The `SPLIT_LINE` command represents the operation normally triggered by pressing Enter.

Conceptually:

```text
SPLIT_LINE
     │
     ▼
Command Engine
     │
     ▼
Document
     │
     ├── Current line is split
     ├── New line is created
     └── Cursor is positioned appropriately
```

Splitting a line affects both the Document and the Cursor.

The Document is responsible for maintaining its line representation, while the Cursor is responsible for the resulting logical position.

Again, the Command Engine coordinates the operation without becoming responsible for the internal mechanics of line splitting.

## Editing Commands and the Modified State

All commands that change the Document have an important additional consequence: the document becomes modified.

For example:

```text
INSERT_CHARACTER
       │
       ▼
Document changes
       │
       ▼
Dirty = true
```

The same applies to deletion and line splitting.

This allows the editor to distinguish between:

```text
Document
    │
    ├── Unmodified
    │
    └── Modified
```

The modified state is not itself part of the text representation. It describes the relationship between the current in-memory Document and its last saved state.

The Command Engine therefore has to coordinate this state transition whenever a command successfully changes the Document.

## Editing Commands Are State Transitions

It is useful to view editing commands in terms of their overall effect rather than their individual algorithms.

```text
INSERT_CHARACTER
       │
       ├── Document changes
       ├── Cursor changes
       └── Dirty state changes
```

```text
DELETE_CHARACTER
       │
       ├── Document changes
       ├── Cursor may change
       └── Dirty state changes
```

```text
SPLIT_LINE
       │
       ├── Document changes
       ├── Cursor changes
       └── Dirty state changes
```

The exact combination depends on the operation and the current document state.

The Command Engine's role is to ensure that the appropriate components participate in the transition.

## Why the Command Engine Should Not Own Editing Algorithms

It may initially seem simpler to put the entire implementation of an editing command inside the Command Engine:

```text
COMMAND_DELETE
     │
     ▼
command_delete()
     │
     ├── manipulate lines
     ├── move cursor
     └── mark document dirty
```

This approach would work for a small program, but it would blur the architectural boundaries established earlier.

The Document already owns the representation of text. The Cursor already owns cursor behavior. Moving their algorithms into the Command Engine would give that module knowledge of internal implementation details belonging to other components.

Instead:

```text
Command Engine
      │
      │ coordinates
      ▼
+-------------+       +-------------+
|  Document   |       |   Cursor    |
| owns text   |       | owns cursor |
+-------------+       +-------------+
```

This keeps each component cohesive and prevents the Command Engine from becoming a large collection of unrelated low-level operations.

## Relationship to Earlier Chapters

The responsibilities can therefore be separated cleanly:

| Concern                              | Responsible Component |
| ------------------------------------ | --------------------- |
| How text is represented              | Document              |
| How text is inserted or removed      | Document              |
| How cursor movement works            | Cursor                |
| Which input produces a command       | Input Engine          |
| Which operation the command requests | Command               |
| How the operation is coordinated     | Command Engine        |
| How the resulting state is displayed | Renderer              |

This division prevents Chapter 16 from duplicating the detailed rules already established in Chapters 13 and 14.

Chapter 13 answered:

> **How is editable text represented and modified?**

Chapter 14 answered:

> **How does the cursor move through that text?**

Chapter 15 answered:

> **How does input become a command?**

Chapter 16 now answers:

> **How does that command coordinate the appropriate operations?**

That progression is important because each layer adds a new responsibility without redefining the responsibilities below it.

## A Complete Editing Example

Consider the user pressing a key to insert the character `A`.

The complete flow is:

```text
Key Press
    │
    ▼
Input Event
    │
    ▼
Input Engine
    │
    │ maps
    ▼
INSERT_CHARACTER('A')
    │
    ▼
Command Engine
    │
    ├──► Document
    │       │
    │       └── insert 'A'
    │
    ├──► Cursor
    │       │
    │       └── update position
    │
    └──► Editor State
            │
            └── mark modified
                    │
                    ▼
                 Renderer
```

Notice that the Command Engine does not need to know the internal data structure of the Document or the detailed cursor movement rules.

It only needs to coordinate the operation through the appropriate interfaces.

This is the key architectural boundary:

> **Commands express what operation should occur; core components provide the rules for carrying out that operation.**

The result is a Command Engine that remains relatively small even as the editor's editing behavior becomes more sophisticated.

## Summary

Editing commands, such as `INSERT_CHARACTER`, `DELETE_CHARACTER`, and `SPLIT_LINE`, are where the Command Engine coordinates operations that change the Document. The Command Engine does not define how text editing works; the Document and Cursor apply their own rules while the Command Engine ensures the affected components participate in one consistent state transition.

Because every editing command modifies the Document, it also marks the document as modified. This keeps the Command Engine focused on coordination rather than duplicating the editing algorithms established in earlier chapters.

Now that the core editing commands are understood, the next section examines how the Command Engine relates to state ownership and its own boundaries.

Continue to **Section 16.7, Command Boundaries and State Ownership**.
