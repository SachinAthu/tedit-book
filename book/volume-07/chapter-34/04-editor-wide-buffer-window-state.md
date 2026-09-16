# 34.4 Editor-Wide State vs Buffer State vs Window State

With multiple buffers, simply adding more `Document` objects is not enough.

The editor now contains several kinds of state with different lifetimes and responsibilities. Some state belongs to the entire editor session. Some belongs to one buffer. Other state belongs to a particular window or view.

If these scopes are mixed together, the architecture becomes difficult to reason about.

A useful model is:

```text
Editor
├── Editor-wide State
│
├── Buffers
│   ├── Buffer A
│   ├── Buffer B
│   └── Buffer C
│
└── Windows
    ├── Window A
    ├── Window B
    └── Window C
```

The central question is:

> **Who should own this piece of state, and how long should it live?**

## Three Different Scopes

The multi-buffer editor can be divided into three primary scopes.

| Scope           | Meaning                        | Examples                                              |
| --------------- | ------------------------------ | ----------------------------------------------------- |
| Editor-wide     | Shared by the editor session   | Configuration, terminal dimensions, buffer collection |
| Buffer-specific | Belongs to one editing context | Document, file association, dirty state, undo history |
| Window-specific | Describes one visual view      | Viewport, window geometry, active view state          |

This classification is more important than the exact structures used to implement it.

The same principle used elsewhere in the architecture still applies:

> **State should have one clear owner.**

## Editor-Wide State

Editor-wide state describes the editor session as a whole.

It should not change merely because the user switches buffers.

Examples include:

```text
Editor-wide State
├── Buffer collection
├── Active tab
├── Tab collection
├── Window layout
├── Terminal dimensions
└── Configuration
```

Note that no "Active Buffer" appears here. The active buffer is not an editor-wide field: it is derived from the window structure — the buffer referenced by the active Window in the active Tab. Storing it as an editor-wide value would create a second source of truth that could fall out of sync with what the active Window displays.

Consider configuration.

A key binding such as a command for saving should normally mean the same thing regardless of which buffer is active. The configuration therefore belongs to the editor rather than to an individual buffer.

Likewise, the terminal dimensions describe the available terminal surface. They are not properties of `main.c` or `editor.c`.

The buffer collection itself is also editor-wide because it describes which editing contexts currently exist.

## Buffer-Specific State

Buffer-specific state follows the editing context.

A simplified model is:

```text
Buffer A
├── Document
├── File association
├── Dirty state
├── Undo history
└── Document-specific derived state
```

Buffer B has its own corresponding state:

```text
Buffer B
├── Document
├── File association
├── Dirty state
├── Undo history
└── Document-specific derived state
```

Switching buffers therefore changes which buffer-specific state is being operated on.

For example:

```text
Active Buffer
      │
      ▼
Buffer A
      │
      ├── Document A
      ├── Undo history A
      └── Dirty state A
```

After switching:

```text
Active Buffer
      │
      ▼
Buffer B
      │
      ├── Document B
      ├── Undo history B
      └── Dirty state B
```

The editor has not created a new undo system or Dirty Flag. It has simply changed the active editing context.

## Window-Specific State

Window-specific state is different.

A window describes **where and how a buffer is being displayed**.

For example:

```text
Window A
├── position
├── width
├── height
├── viewport
└── displayed Buffer
```

The viewport is particularly important.

Earlier, the viewport represented the portion of the document currently visible on the terminal. With multiple windows, there can now be several visible regions at the same time.

For example:

```text
┌──────────────────────┬──────────────────────┐
│ Window A             │ Window B             │
│                      │                      │
│ main.c               │ editor.c             │
│ line 10              │ line 25              │
│ line 11              │ line 26              │
│ line 12              │ line 27              │
│                      │                      │
└──────────────────────┴──────────────────────┘
```

Each window needs to know which portion of its buffer it is displaying.

Therefore, viewport state cannot remain a single editor-wide object.

Conceptually:

```text
Window A → Viewport A
Window B → Viewport B
```

This allows two windows to display different portions of the same buffer without interfering with each other.

## The Cursor Requires a More Careful Decision

The cursor is more subtle.

In the original single-document editor, there was one cursor:

```text
Editor State
└── Cursor
```

With multiple buffers, a simple extension could make the cursor buffer-specific:

```text
Buffer A → Cursor A
Buffer B → Cursor B
```

This works well when each buffer has one editing position.

However, multiple windows introduce another possibility:

```text
Window A ──┐
           ├── Buffer A
Window B ──┘
```

If both windows display the same buffer, should they necessarily share the same cursor position?

Not always.

A more flexible model allows each window to maintain its own editing position:

```text
Window A → Buffer A → Cursor A
Window B → Buffer A → Cursor B
```

Both windows still reference the same document, but each view can have an independent position.

This distinction is important:

```text
Document position
        ≠
Window viewing position
```

The document remains shared. The way the user is currently viewing and interacting with it can differ.

For the multi-window architecture, cursor state is therefore best associated with the **view/window editing context**, rather than treating one cursor as an intrinsic property of the document.

This is an architectural evolution from the single-document model.

## Buffer and Window Form a Relationship

A useful way to think about the distinction is:

```text
Buffer
    answers:
    "What am I editing?"

Window
    answers:
    "Where and from what position am I viewing it?"
```

The relationship can then be represented as:

```text
Window
├── Geometry
├── Viewport
├── Cursor
└── Buffer reference
          │
          ▼
       Buffer
       ├── Document
       ├── File association
       ├── Dirty state
       ├── Undo history
       └── Derived document state
```

This separation prevents visual state from leaking into the document itself.

## One Buffer, Multiple Windows

Consider a document that is large enough that two windows display different regions of it:

```text
              Buffer A
                  │
          ┌───────┴───────┐
          │               │
      Window A         Window B
      lines 1–20       lines 80–100
```

Both windows reference the same buffer.

They must therefore see the same underlying document changes.

If text is inserted into Buffer A through Window A, Window B should eventually render the updated document as well.

But their view state can remain different:

```text
Window A
    viewport = top of document
    cursor   = near line 10

Window B
    viewport = lower portion
    cursor   = near line 90
```

This is one of the strongest reasons to separate buffers from windows.

If the viewport were stored directly in the document, the two windows could not independently choose what portion to display.

## A Practical Ownership Model

The resulting ownership model can be summarized as:

```text
EDITOR-WIDE
────────────────────────────
Configuration
Terminal dimensions
Buffer collection
Tab collection
Window/layout structure


BUFFER-SPECIFIC
────────────────────────────
Document
File association
Dirty state
Undo history
Document-specific derived state


WINDOW-SPECIFIC
────────────────────────────
Window geometry
Viewport
Cursor position
Displayed buffer
```

This does not mean that every implementation must use exactly these structures. The important part is the ownership boundary.

Taken together, the Window-specific values constitute the Window's **View** of its Buffer: the Buffer reference binds the presentation to an editing context, while the Viewport and Cursor are the view state. The View names that presentation binding; the Window owns it (Section 34.2).

A structure can change without changing the principle.

## Why Ownership Matters

Poor ownership produces subtle bugs.

Suppose the viewport is stored globally:

```text
Editor
└── Viewport
```

Switching between windows would force all windows to share one viewing position.

Suppose the Dirty Flag is also global:

```text
Editor
└── Dirty
```

The editor could no longer accurately represent which buffer contains unsaved changes.

Suppose configuration is stored separately in every buffer:

```text
Buffer A → Configuration A
Buffer B → Configuration B
Buffer C → Configuration C
```

The same editor session could then have different key bindings depending on which buffer is active, unless that behavior were explicitly intended.

Each of these designs gives a state object the wrong lifetime.

## The Lifetime Test

A simple test helps identify the correct scope:

**If the user switches buffers, should this state change?**

If not, it is probably not buffer-specific.

**If the user opens another window showing the same buffer, should this state be independent?**

If yes, it is probably window-specific.

**If neither switching buffers nor changing windows should alter it, it may be editor-wide.**

This produces a useful decision process:

```text
                    Does it describe
                    the whole session?
                         │
                    ┌────┴────┐
                   Yes       No
                    │         │
              Editor-wide     ▼
                         Does each window
                         need its own value?
                              │
                         ┌────┴────┐
                        Yes       No
                         │         │
                    Window-specific ▼
                                  Buffer-specific
```

This is not a replacement for architectural judgment, but it is a useful starting point.

## The Resulting Mental Model

The multi-buffer editor is therefore no longer best understood as:

```text
Editor State
├── Document
├── Cursor
└── Viewport
```

Instead, it becomes a set of related scopes:

```text
Editor
│
├── Editor-wide State
│
├── Buffer Collection
│   ├── Buffer A
│   │   └── Document A
│   ├── Buffer B
│   │   └── Document B
│   └── Buffer C
│       └── Document C
│
└── Window Collection
    ├── Window A
    │   ├── Buffer reference
    │   ├── Cursor
    │   └── Viewport
    └── Window B
        ├── Buffer reference
        ├── Cursor
        └── Viewport
```

The important architectural change is not simply that there are now more objects.

It is that **state is now organized according to ownership and lifetime**.

Once that boundary is clear, the editor can safely introduce operations such as creating buffers, switching buffers, displaying one buffer in several windows, and managing tabs without turning the entire system into shared mutable state.

The next challenge is to manage the collection of buffers itself. The editor needs a component responsible for creating, tracking, locating, switching, and eventually destroying buffers without making unrelated modules responsible for that lifecycle.

## Summary

State is organized into three scopes with distinct lifetimes: editor-wide state (configuration, terminal dimensions, collections), buffer-specific state (document, file association, dirty state, undo history), and window-specific state (geometry, viewport, cursor). The lifetime test — whether switching buffers would change a value, or whether a second window would need its own — decides ownership and prevents shared documents, independent views, and global policy from leaking into one another.

The next section introduces the component responsible for managing the buffer collection itself: **Section 34.5, The Buffer Manager**.
