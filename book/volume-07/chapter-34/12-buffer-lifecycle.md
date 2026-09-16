# 34.12 Buffer Lifecycle

A Buffer is not simply created when a file is opened and destroyed when the user switches away from it. It has a lifecycle that begins when the editing context is created and ends only when that context is explicitly closed and no longer needed.

Understanding this lifecycle is important because different stages have different ownership and safety requirements.

A simplified lifecycle is:

```text
New
 ↓
Empty Buffer
 ↓
Editing
 ↓
Modified
 ↓
Saved
 ↓
Closed
```

A file-backed Buffer follows a slightly different creation path:

```text
Open File
    ↓
Create Buffer
    ↓
Load Document
    ↓
Associate File
    ↓
Buffer Ready
```

The lifecycle is therefore about the **editing context**, not merely the file.

## Creating a Buffer

A Buffer may be created in several ways.

The most common cases are:

- the user creates a new unnamed editing context
- the user opens an existing file
- the editor creates an editing context as part of startup

For an unnamed Buffer, creation can be conceptually represented as:

```text
Create Buffer
     │
     ▼
Create Document
     │
     ▼
Initialize Buffer State
     │
     ▼
Empty Buffer Ready
```

The resulting Buffer has a valid Document even though it has no associated file.

For a file-backed Buffer:

```text
Create Buffer
     │
     ▼
Load File
     │
     ▼
Create Document Contents
     │
     ▼
Associate File
     │
     ▼
Buffer Ready
```

The important point is that Buffer creation establishes an **editing context**. Loading a file is one way of initializing that context, but it is not the definition of a Buffer.

## The Empty and Initial States

A newly created Buffer begins in a clean state.

For an unnamed Buffer:

```text
Buffer
 ├── Document → empty
 ├── File → none
 └── Dirty → false
```

For a file-backed Buffer:

```text
Buffer
 ├── Document → loaded contents
 ├── File → associated file
 └── Dirty → false
```

In both cases, the Buffer is ready for editing.

Once the user changes its Document, the Buffer becomes modified:

```text
Editing
   │
   ▼
Document changes
   │
   ▼
Buffer becomes dirty
```

The dirty state belongs to the Buffer because it describes whether that editing context contains changes that have not been successfully persisted.

This allows different Buffers to have independent states:

```text
Buffer A → clean
Buffer B → modified
Buffer C → clean
```

Switching between them does not change those states.

## Editing and Modification

A Buffer enters the modified state when an editing operation changes its Document.

For example:

```text
Buffer A
    │
    ▼
Insert character
    │
    ▼
Document changes
    │
    ▼
Dirty = true
```

The Buffer remains modified until the relevant persistence operation succeeds.

This distinction is important because a command that _attempts_ to save a Buffer is not necessarily a command that _successfully_ saves it.

The state transition is therefore:

```text
Modified
    │
    │ successful save
    ▼
Saved / Clean
```

but not:

```text
Modified
    │
    │ save attempted
    ▼
Clean
```

A failed save must leave the Buffer modified.

## Switching Does Not End the Lifecycle

A Buffer can stop being displayed in the active Window without being closed.

For example:

```text
Window
   │
   ▼
Buffer A
```

After switching:

```text
Window
   │
   ▼
Buffer B
```

Buffer A still exists.

Its Document, file association, dirty state, undo history, and other Buffer-specific state remain intact.

This distinction is fundamental:

```text
Not currently displayed
        ≠
Closed
```

A Buffer may therefore move through periods where it is not the active editing context while continuing to exist in the editor's Buffer collection.

When the user returns to it, the existing editing context is restored rather than reconstructed from the file system.

## Closing a Buffer

Closing a Buffer is different from switching away from it.

Conceptually:

```text
Active Buffer
      │
      ▼
Close Request
      │
      ▼
Check Buffer State
      │
      ├── Clean ──────► Close
      │
      └── Modified ──► Protect / Confirm
```

A clean Buffer can normally be closed without additional persistence work.

A modified Buffer requires protection because closing it could discard changes that have not been saved.

The editor therefore needs to distinguish at least these cases:

```text
Clean Buffer
    → safe to destroy

Modified Buffer
    → must not be silently destroyed
```

The exact user interaction for protecting a modified Buffer belongs to the command and user-interface design. The architectural requirement is that the Buffer's modified state must be consulted before destruction.

## Modified-Buffer Protection

Suppose the editor contains:

```text
Buffer A → main.c    → clean
Buffer B → editor.c  → modified
```

If the user attempts to close Buffer B, the editor must not simply remove it from the Buffer collection.

Instead, the operation must recognize that unsaved state exists:

```text
Close Buffer B
      │
      ▼
Is Buffer modified?
      │
      ├── No  → destroy
      │
      └── Yes → require appropriate protection
```

Protection may ultimately involve saving, cancelling the close operation, or another explicitly defined user decision.

What matters architecturally is that **destruction is conditional on the Buffer's state**.

The Buffer Manager owns the Buffer's lifecycle, but it should not silently discard information merely because a destruction request was received.

## Buffer Destruction

Once a Buffer has been safely closed, its resources can be released.

Conceptually:

```text
Closed Buffer
     │
     ▼
Release Buffer-owned resources
     │
     ├── Document
     ├── Undo history
     ├── File association
     └── Buffer-specific derived state
     │
     ▼
Buffer destroyed
```

The exact order and implementation details depend on the ownership relationships established by the architecture.

The important principle is that destruction belongs to the lifecycle owner.

The Buffer Manager is responsible for managing the collection and lifetime of Buffers, while the individual components remain responsible for their own internal resources.

This prevents unrelated components from independently deciding when a Buffer should disappear.

## The Last Buffer

The editor also needs a defined policy for what happens when the user attempts to close the final remaining Buffer.

For example, the editor could maintain an invariant such as:

```text
At least one Buffer always exists
```

In that model, closing the last Buffer would create or retain an empty unnamed Buffer.

Alternatively, the architecture could permit the Buffer collection to become empty and handle the empty-editor state separately.

The important point is that this is a **lifecycle policy**, not an accidental consequence of implementation.

A well-defined policy prevents commands from encountering ambiguous states such as:

```text
Active Window → no Buffer
```

when the rest of the editor assumes that an active editing context always exists.

The chosen rule should therefore be explicit in the editor's state model.

## Buffers That Are No Longer Displayed

A Buffer may remain alive even when no Window currently displays it.

For example:

```text
Buffer A ──► Document A
Buffer B ──► Document B
Buffer C ──► Document C

Window 1 ──► Buffer A
Window 2 ──► Buffer C
```

Buffer B is not currently displayed, but it remains part of the editor's Buffer collection.

This is useful because organizational structures and editing contexts are separate from visual presentation.

A Buffer should be destroyed because its lifecycle has ended—not merely because no Window currently happens to reference it.

If the architecture later introduces automatic cleanup of unused Buffers, that would be a separate lifecycle policy and would need to account for unsaved changes and other retained state.

## Lifecycle Ownership

The lifecycle can now be understood in terms of ownership:

```text
Editor
  │
  └── Buffer Manager
        │
        ├── Buffer A
        ├── Buffer B
        └── Buffer C
```

The Buffer Manager controls the collection and lifetime of these Buffers.

Each Buffer, in turn, represents an editing context containing or referencing the state required for that context.

Other components interact with the Buffer but do not independently decide its lifetime.

For example:

```text
Window
   │
   └── references Buffer

Renderer
   │
   └── reads Buffer state

Command Engine
   │
   └── requests operations on Buffer

File Manager
   │
   └── persists Buffer's Document
```

None of these relationships mean that those components own the Buffer's lifecycle.

This preserves the principle established earlier:

> **One owner, many readers.**

There should be one authoritative mechanism responsible for deciding when a Buffer exists and when it is destroyed.

## A Complete Lifecycle

Putting the stages together gives us a more complete picture:

```text
                 ┌──────────────┐
                 │ Create Buffer│
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │     Ready    │
                 └──────┬───────┘
                        │
                  edit  │
                        ▼
                 ┌──────────────┐
                 │   Modified   │
                 └──────┬───────┘
                        │
                  save  │
                        ▼
                 ┌──────────────┐
                 │ Clean/Saved  │
                 └──────┬───────┘
                        │
                  close │
                        ▼
                 ┌──────────────┐
                 │    Closed    │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │   Destroyed  │
                 └──────────────┘
```

A Buffer can also move repeatedly between the clean and modified states:

```text
Clean
  │
 edit
  ▼
Modified
  │
 save
  ▼
Clean
  │
 edit
  ▼
Modified
```

Switching between Buffers does not appear as a lifecycle transition because it does not change whether the Buffer exists.

```text
Buffer A ── active ──► inactive
                     │
                     ▼
                 still alive
```

That distinction keeps **visibility, activity, and lifetime** from becoming accidentally coupled.

## Engineering Insight

The Buffer lifecycle reveals why a Buffer is more than a wrapper around a Document.

It is a persistent editing context with its own state and lifetime.

A Buffer may be created without a file, associated with a file, modified, saved, temporarily not displayed, returned to later, and eventually destroyed.

The architecture therefore needs to preserve three separate concepts:

```text
Activity
   → Is this Buffer currently being edited?

Visibility
   → Is this Buffer currently displayed?

Lifetime
   → Does this Buffer still exist?
```

These are related, but they are not the same thing.

Keeping them separate prevents buffer switching, window management, file persistence, and resource destruction from becoming one tangled operation.

The result is a lifecycle that remains understandable as the editor grows:

```text
Create
  ↓
Use
  ↓
Modify
  ↓
Save
  ↓
Switch / Return
  ↓
Close
  ↓
Destroy
```

Each transition has a clear reason, a clear owner, and a clear relationship to the rest of the architecture.

## Summary

A buffer is a persistent editing context with its own lifecycle: created, edited until dirty, saved clean, switched away from without closing, protected before destruction, and eventually destroyed by the Buffer Manager. Activity, visibility, and lifetime are separate concepts, and destruction must account for unsaved state.

The next section revisits the editor's existing features and assigns each its correct scope: **Section 34.13, Scope of Existing Editor Features**.
