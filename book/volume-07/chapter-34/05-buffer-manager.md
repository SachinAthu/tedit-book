# 34.5 The Buffer Manager

Once an editor can contain multiple buffers, something must keep track of them.

The editor should not require every command, input handler, renderer, and file operation to maintain its own list of buffers. Doing so would distribute ownership of the same collection across unrelated modules and quickly create inconsistent state.

Instead, the multi-buffer architecture introduces a dedicated **Buffer Manager**.

Its responsibility is to manage the collection and lifecycle of buffers.

Conceptually:

```text
Editor
  │
  └── Buffer Manager
       ├── Buffer A
       ├── Buffer B
       └── Buffer C
```

The Buffer Manager does not own the contents of the documents in the conceptual sense. Each buffer remains responsible for its own editing context.

The manager owns the **collection and organization of those buffers**.

## Why a Separate Manager?

A collection of buffers creates several responsibilities that did not exist in a single-document editor.

The system must be able to:

- create a buffer,
- register it,
- locate it,
- identify the active buffer,
- switch to another buffer,
- remove a buffer,
- determine whether buffers still exist,
- and eventually coordinate lifecycle decisions.

Without a dedicated boundary, these responsibilities tend to spread throughout the application.

For example, a command might directly manipulate an array of buffers:

```text
Save Command
    └── accesses buffer array

Switch Command
    └── accesses buffer array

Close Command
    └── accesses buffer array

Renderer
    └── accesses buffer array
```

Now several modules understand how buffers are stored.

If the representation changes, all of them may need to change.

A manager provides a narrower boundary:

```text
Command Engine
      │
      ▼
Buffer Manager
      │
      ├── Buffer A
      ├── Buffer B
      └── Buffer C
```

The rest of the architecture can ask for buffer-related operations without needing to know how the collection itself is represented.

## Managing a Collection, Not Editing Text

The Buffer Manager should not become another version of the Command Engine.

It should not decide what inserting a character means.

It should not implement cursor movement.

It should not perform rendering.

It should not contain file parsing logic.

Its concern is the **existence and organization of buffers**.

A useful conceptual boundary is:

```text
Buffer Manager
    "Which buffers exist?"

Command Engine
    "What operation should be performed?"

Buffer
    "What editing context does this document have?"

Document
    "What text exists?"
```

These questions are related, but they are different responsibilities.

## Buffer Identity

Each buffer needs a stable identity within the editor.

A filename is not sufficient.

Consider:

```text
Buffer 1 → /project/main.c
Buffer 2 → /project/editor.c
Buffer 3 → unnamed
```

The third buffer has no file path at all, but it is still a distinct buffer.

Even when a buffer has a filename, the filename describes its file association rather than necessarily serving as its internal identity.

A manager can therefore conceptually maintain:

```text
Buffer ID 1 → Buffer A
Buffer ID 2 → Buffer B
Buffer ID 3 → Buffer C
```

The exact identifier could be an integer, pointer, handle, or another implementation-specific mechanism.

The architectural requirement is simply that buffers can be distinguished reliably.

## The Active Buffer

At any moment, commands normally operate on one active editing context.

For example:

```text
Buffer Manager
├── Buffer A
├── Buffer B  ← active
└── Buffer C
```

The active buffer is therefore a relationship between the editor and its buffer collection.

Changing the active buffer does not destroy the previous one.

It simply changes which buffer subsequent editing commands target.

```text
Before:

Active → Buffer A


Switch


After:

Active → Buffer B
```

Buffer A remains alive:

```text
Buffer Manager
├── Buffer A
├── Buffer B  ← active
└── Buffer C
```

This distinction is fundamental.

**Switching is not creation.**

**Switching is not destruction.**

It is a change in which existing editing context is currently selected.

## Finding a Buffer

The manager may also need to locate a buffer based on some identity or association.

Possible lookup information includes:

```text
buffer ID
file association
buffer index
```

The exact lookup strategy depends on the chosen collection structure.

At this level, the important point is that lookup belongs behind the Buffer Manager boundary.

A command should conceptually request:

```text
find buffer X
```

rather than knowing whether buffers are stored in an array, linked list, hash table, or another structure.

This keeps the collection representation replaceable.

## Creating a Buffer

Creating a buffer is also a lifecycle operation.

A new buffer might initially contain an empty document:

```text
Create Buffer
      │
      ▼
┌───────────────┐
│ New Buffer    │
│               │
│ Document: ""  │
│ File: none    │
│ Dirty: false  │
└───────────────┘
```

Alternatively, a buffer may be created as part of opening a file:

```text
Open File
    │
    ▼
Create Buffer
    │
    ▼
Load Document
    │
    ▼
Associate File
```

The File Manager remains responsible for persistence operations.

The Buffer Manager is responsible for the existence of the editing context that receives the loaded document.

This distinction prevents file I/O responsibilities from leaking into buffer collection management.

## Removing a Buffer

Removing a buffer is more complicated than removing an element from a collection.

A buffer may contain unsaved changes:

```text
Buffer A
    Dirty = true
```

Removing it could therefore discard user data.

The Buffer Manager should participate in the lifecycle operation, but it should not silently decide that unsaved work can be destroyed.

Instead, closing a buffer should involve a decision:

```text
Close Buffer
     │
     ▼
Is it modified?
   /       \
 No         Yes
 │           │
 ▼           ▼
Close     Confirm / save / cancel
```

The exact user interaction belongs to the command and application layers.

The important architectural principle is that **buffer destruction is a lifecycle boundary** and must account for the state owned by that buffer.

## The Buffer Manager Does Not Own the Buffer's Internal State

There is an important distinction between **owning the collection** and **owning every field inside each buffer**.

The manager may own the lifetime of:

```text
Buffer A
Buffer B
Buffer C
```

while the buffer itself contains:

```text
Buffer A
├── Document
├── File association
├── Dirty state
└── Undo history
```

The manager should not reach into these fields simply because it can.

For example, this would create poor coupling:

```text
Buffer Manager
    └── modifies Buffer.undo_history
```

unless that operation is explicitly part of its responsibility.

A healthier relationship is:

```text
Buffer Manager
    │
    └── manages Buffer lifetime

Buffer
    │
    └── owns its editing state
```

Ownership should therefore exist at more than one level.

## Collection Ownership vs State Ownership

This distinction can be summarized as:

```text
Editor
  │
  └── Buffer Manager
        │
        ├── owns collection
        │
        ├── manages lifetime
        │
        └── references
             │
             ├── Buffer A
             │     ├── Document
             │     ├── Dirty state
             │     └── Undo history
             │
             └── Buffer B
                   ├── Document
                   ├── Dirty state
                   └── Undo history
```

The manager answers:

> Which buffers exist and how are they managed?

The buffer answers:

> What state belongs to this editing context?

This separation prevents the manager from becoming a giant state-management object.

## Interaction with Other Components

The Buffer Manager sits between the collection of editing contexts and the components that need to access them.

A simplified relationship is:

```text
             Command Engine
                    │
                    ▼
             Buffer Manager
                    │
             ┌──────┼──────┐
             ▼      ▼      ▼
          Buffer  Buffer  Buffer
             │      │      │
             ▼      ▼      ▼
         Document Document Document
```

The renderer may need access to the active buffer, while commands may need to switch or close buffers.

The File Manager may load or save the document associated with a buffer.

None of these relationships require those modules to own the collection themselves.

The manager provides the organizational boundary.

## The Manager Should Remain Small

A common architectural mistake is to make a manager responsible for everything related to the objects it manages.

That produces structures such as:

```text
Buffer Manager
├── buffer collection
├── file loading
├── file saving
├── cursor movement
├── undo
├── search
├── syntax highlighting
├── rendering
└── configuration
```

The name has changed, but the architectural problem remains.

A manager should not absorb unrelated responsibilities merely because it has access to the objects involved.

The Buffer Manager should remain focused on **buffer collection and lifecycle management**.

## Collection Representation Is an Implementation Detail

The manager also provides an architectural opportunity.

The editor does not need to commit at this level to one particular collection structure.

For a small editor, a dynamic array may be sufficient:

```text
[ Buffer A ][ Buffer B ][ Buffer C ]
```

A linked structure could also represent the collection:

```text
Buffer A → Buffer B → Buffer C
```

A more sophisticated implementation might use identifiers and indexed lookup.

The choice should depend on actual requirements such as:

- number of buffers,
- lookup frequency,
- insertion and removal patterns,
- memory behavior,
- cache locality,
- implementation complexity.

The important architectural point is that other modules should depend on the **Buffer Manager interface**, rather than directly depending on the collection representation.

The underlying data structure can therefore evolve independently.

## A Narrow Responsibility

The Buffer Manager can ultimately be summarized by four questions:

```text
1. Which buffers exist?
2. How are buffers identified?
3. Which buffer is active?
4. How are buffers created and destroyed?
```

Everything else should be evaluated carefully before being added to its responsibility.

This keeps the architecture aligned with the broader principle of high cohesion and low coupling.

The introduction of multiple buffers therefore does not require a giant new subsystem.

It requires a clear ownership boundary around a collection of independently managed editing contexts.

The next step is to examine **switching between buffers**. Creating and storing multiple buffers has little value unless the user can move between them while preserving the state of each editing context.

## Summary

The Buffer Manager is a narrow ownership boundary around the buffer collection and its lifecycle: which buffers exist, how they are identified, which is active, and how they are created and destroyed. It owns the collection, not the internal editing state of each buffer, and it must not absorb unrelated responsibilities such as file I/O, rendering, or undo.

The next section examines the operation that gives those buffers their value: **Section 34.6, Switching Between Buffers**.
