# 34.3 What Belongs to a Buffer?

Now that the terms **Document**, **Buffer**, **View**, **Window**, and **Tab** are separated, the next question is more important than simply deciding where to store a few pointers:

> **What information must remain associated with a particular editing context?**

A buffer should preserve everything necessary for the editor to continue working with one document as an independent editing context.

This means that switching from one buffer to another should not destroy the state of either one.

Conceptually, a buffer can be viewed as a container around a document and the state that belongs to editing that document:

```text
Buffer
├── Document
├── File association
├── Modified / saved state
├── Undo history
├── Search-related state
├── Syntax-analysis state
└── Other document-specific editing state
```

The exact implementation can evolve, but the ownership principle is important.

## The Document

The most obvious component of a buffer is its `Document`.

The document contains the actual editable text. The buffer provides the larger context in which that text is being edited.

This distinction becomes useful because a buffer has an identity independent of the text currently stored in it.

For example, consider two buffers:

```text
Buffer A → Document A → main.c
Buffer B → Document B → editor.c
```

Both documents may contain similar text, or even identical text, but they are still different editing contexts.

The buffer identity should therefore not be derived simply from the contents of the document.

## File Association

A buffer also needs to know whether its document is associated with a file.

For example:

```text
Buffer A
    Document: main.c
    File: /project/src/main.c

Buffer B
    Document: untitled
    File: none
```

An unnamed document is still a valid buffer.

This distinction matters because creating a new document does not necessarily mean creating a file immediately. A user can begin editing an unnamed buffer and decide later where it should be saved.

The file association therefore describes the relationship between the buffer and persistent storage. It is not the identity of the buffer itself.

## Modified State

The modified, or dirty, state must also belong to the individual buffer.

Suppose the editor contains three buffers:

```text
Buffer A → modified
Buffer B → saved
Buffer C → modified
```

If the user switches from Buffer A to Buffer B, Buffer A must remain marked as modified.

A single editor-wide Dirty Flag would not be sufficient:

```text
Editor
└── dirty = true
```

There would be no way to determine which document needs saving.

Instead, the state must conceptually exist per buffer:

```text
Buffer A → dirty = true
Buffer B → dirty = false
Buffer C → dirty = true
```

This is an example of a broader rule:

> **State that describes whether a particular document has changed belongs to that document's editing context.**

The exact definition of the dirty state remains the same as established earlier. Chapter 34 changes its **scope**, not its meaning.

## Undo History

Undo history is another example of state that cannot be shared indiscriminately between buffers.

Consider:

```text
Buffer A
    edits:
        insert "hello"
        delete "x"

Buffer B
    edits:
        insert "world"
```

If the user switches from Buffer A to Buffer B and presses Undo, the editor must undo an operation belonging to Buffer B.

It must not accidentally undo the most recent operation performed on Buffer A.

Therefore, undo history is associated with the buffer's editing context.

Conceptually:

```text
Buffer A → Undo History A
Buffer B → Undo History B
Buffer C → Undo History C
```

The internal undo mechanism does not need to be redesigned here. The important architectural observation is that its lifetime follows the buffer.

## Search-Related State

Search can also have state associated with the current editing context.

For example, a search session may contain information such as:

```text
query
direction
current match
active state
```

When working with multiple buffers, the editor must decide whether this information follows the active buffer, belongs to a particular view, or is shared across the editor.

For the buffer model, the important point is that search results and positions are meaningful relative to a particular document.

A match position in `main.c` has no meaning when applied directly to `editor.c`.

Therefore, any search information that describes a document-specific search context must have an appropriate association with that buffer.

This does not mean that every search feature must permanently live inside the buffer. It means that its ownership must respect the scope of the information it represents.

## Syntax-Analysis State

Syntax highlighting introduces another useful example.

The syntax information derived from a document must correspond to that document.

Conceptually:

```text
Buffer A
    Document A
        Syntax Analysis A

Buffer B
    Document B
        Syntax Analysis B
```

The syntax analysis should never become part of the text itself.

If Buffer A is modified, its derived syntax information may need to be updated or invalidated. Buffer B should not be affected merely because Buffer A changed.

Again, Chapter 34 does not redefine how syntax highlighting works. It establishes where the lifetime of its document-specific state belongs when multiple documents exist.

## Buffer Identity Is Not File Identity

It is tempting to assume that one file corresponds to one buffer.

That is often a useful convention, but it is not a fundamental definition of a buffer.

For example, an editor may eventually allow:

```text
Buffer A → /project/main.c
Buffer B → /project/main.c
```

Whether this situation should be allowed, prevented, or resolved by sharing one buffer is a separate architectural decision.

The important distinction is:

```text
File path      → identifies persistent storage
Document       → represents editable text
Buffer         → represents an editing context
```

These concepts are related, but they are not interchangeable.

This distinction becomes especially important when considering multiple views of the same document.

## A Buffer Is Not a Dumping Ground

There is a danger in defining a buffer too broadly.

Once a structure is described as an "editing context," it can become tempting to place every piece of editor state inside it:

```text
Buffer
├── Document
├── Cursor
├── Viewport
├── Terminal dimensions
├── Window position
├── Theme
├── Key bindings
├── Global configuration
└── ...
```

This is poor ownership.

A buffer should not become a convenient container for unrelated state.

For example, terminal dimensions describe the terminal, not a particular document. Key bindings describe editor policy, not one buffer. Window position describes where something is displayed, not what document exists.

The purpose of the buffer boundary is precisely the opposite:

> **A buffer should contain state because that state belongs to the editing context, not merely because the state is needed while editing.**

## Lifetime Is the Real Test

A useful way to determine whether something belongs to a buffer is to ask:

> **What should happen to this state when the user switches to another buffer?**

If the answer is:

> "It should remain unchanged and be restored when this buffer becomes active again."

then the state is a strong candidate for buffer ownership.

For example:

```text
Switch away from Buffer A
        │
        ├── Document A remains
        ├── Dirty state remains
        ├── Undo history remains
        └── Document-specific analysis remains
```

When the user returns:

```text
Switch to Buffer A
        │
        └── Buffer A continues from its previous editing context
```

This is what makes multiple buffers useful rather than simply providing multiple files.

## The Boundary Is Not Yet Complete

At this point, we have identified several kinds of state that naturally follow a buffer.

But an important question remains unresolved.

Consider the following:

```text
Cursor
Viewport
Window position
Tab selection
Terminal size
Configuration
```

Some of these belong to a particular editing context. Others belong to the way that context is displayed. Still others belong to the editor as a whole.

The distinction becomes especially important when **two windows display the same buffer**.

For example:

```text
Window A ──┐
           ├── Buffer A ── Document A
Window B ──┘
```

Should both windows share the same cursor?

Should they share the same viewport?

Should switching windows change the editing position?

These are ownership questions, not rendering details.

The next section will establish a complete state-scope model for the multi-buffer editor:

```text
Editor-wide State
       │
       ├── Buffer-specific State
       │
       └── Window-specific State
```

That classification will give us a precise answer to where the cursor, viewport, tabs, configuration, and other state should live.

## Summary

A Buffer owns the state that describes one editing context: its Document, file association, dirty state, undo history, and document-derived information such as search and syntax state. It must not become a dumping ground for unrelated state; the real test is whether the state has to survive switching away and returning. Cursor, viewport, and other display-oriented state remain unresolved by this ownership principle.

The next section draws the full boundary between the three scopes of state: **Section 34.4, Editor-Wide State vs Buffer State vs Window State**.
