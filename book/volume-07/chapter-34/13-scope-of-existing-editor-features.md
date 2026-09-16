# 34.13 Scope of Existing Editor Features

Introducing multiple Buffers does not require redesigning every existing editor feature. Most features already have a natural owner; the main architectural task is to ensure that each feature continues to operate within the correct scope.

The important question is no longer simply:

> Where does this feature exist?

It is:

> **Which editing context does this feature belong to?**

The answer may be editor-wide, Buffer-specific, or Window-specific.

## Feature Scope

The established ownership model can be summarized as follows:

| Feature         | Scope                               |
| --------------- | ----------------------------------- |
| Document        | Buffer                              |
| Dirty state     | Buffer                              |
| Undo history    | Buffer                              |
| Search context  | Appropriate document/Buffer context |
| Syntax analysis | Buffer/Document                     |
| Configuration   | Editor-wide                         |
| Cursor          | Window                              |
| Viewport        | Window                              |
| Terminal size   | Editor-wide                         |

This distinction prevents state from accidentally becoming global simply because the editor now contains several editing contexts.

## Document State Belongs to the Buffer

A Document represents the logical text being edited.

Because each Buffer represents an editing context, its Document is Buffer-scoped:

```text
Buffer A ──► Document A
Buffer B ──► Document B
Buffer C ──► Document C
```

Changing Buffer A's Document must not modify Buffer B's Document.

This is the basic isolation required for independent editing contexts.

The same principle applies to state derived directly from the Document. Features such as syntax analysis operate on the relevant Buffer's document contents rather than on one global document.

## Dirty State Belongs to the Buffer

The dirty state answers a specific question:

> Does this Buffer contain changes that have not been successfully persisted?

Therefore, dirty state cannot remain global.

For example:

```text
Buffer A → clean
Buffer B → modified
Buffer C → clean
```

Saving Buffer B changes Buffer B's state:

```text
Buffer A → clean
Buffer B → clean
Buffer C → clean
```

The other Buffers are unaffected.

This also means that closing, switching, or saving one Buffer must not accidentally alter the dirty state of another.

## Undo History Belongs to the Buffer

Undo and redo history must follow the editing context whose operations produced it.

Consider:

```text
Buffer A
    │
    └── Undo history A

Buffer B
    │
    └── Undo history B
```

If the user edits Buffer A, switches to Buffer B, and performs an undo, that undo must apply to Buffer B's history—not Buffer A's.

The multi-buffer model therefore reinforces the earlier decision that undo history is Buffer-specific.

The command itself can remain unchanged. What changes is the context from which the command obtains its history.

## Search Context Follows the Appropriate Editing Context

Search introduces a more subtle scope question.

The search operation operates on document content, so search results must be interpreted relative to the relevant Buffer and its Document.

For example:

```text
Buffer A
    └── Document A
         └── Search result

Buffer B
    └── Document B
         └── Search result
```

A search result from Buffer A must not accidentally be interpreted as a position in Buffer B.

The exact persistence and ownership of search state remains a feature-level decision. The important multi-buffer rule is that logical positions must always be interpreted in the context of the correct Document.

This avoids treating a document position as a globally meaningful coordinate.

## Syntax Analysis Is Buffer-Scoped

Syntax analysis is derived from document contents.

With multiple Buffers:

```text
Buffer A
   │
   ├── Document A
   └── Syntax state A

Buffer B
   │
   ├── Document B
   └── Syntax state B
```

The analysis for one Buffer must not be reused blindly for another Buffer whose contents differ.

The Renderer consumes the appropriate derived state when rendering the Buffer.

This preserves the established separation:

```text
Document
   │
   ▼
Derived syntax state
   │
   ▼
Renderer
```

The Document itself still does not store visual styling information.

## Cursor and Viewport Belong to the Window

The most important scope change occurs with visual editing state.

A Cursor belongs to a Window because two Windows may display the same Buffer while maintaining independent editing positions.

For example:

```text
Window A ──► Buffer A
    │
    └── Cursor A

Window B ──► Buffer A
    │
    └── Cursor B
```

Both Windows refer to the same Document, but their Cursors may differ.

The same applies to the Viewport:

```text
Window A ──► Viewport A
Window B ──► Viewport B
```

This allows two Windows to show different regions of the same Buffer.

Therefore, neither Cursor nor Viewport should be moved into the Buffer merely because the Buffer contains the text being displayed.

The Buffer represents **what is being edited**.

The Window represents **how and where that editing context is currently viewed**.

Together with the Buffer reference, the Cursor and Viewport form the Window's **View** of the Buffer — the presentation binding defined in Section 34.2.

## Configuration Remains Editor-Wide

Configuration does not become Buffer-specific merely because there are multiple Buffers.

For example:

```text
Editor
   │
   └── Configuration
         ├── tab width
         ├── display preferences
         └── key bindings
```

These values remain policy data belonging to the editor as a whole.

A command may use configuration while operating on a particular Buffer, and the Renderer may use configuration while rendering a particular Window, but that does not transfer ownership of configuration to those components.

This preserves the established rule that configuration is centralized and validated rather than being independently interpreted by every module.

## Terminal Size Remains Editor-Wide

The terminal dimensions describe the available output surface:

```text
Editor
   │
   └── Terminal Size
         ├── rows
         └── columns
```

The dimensions affect all Windows because the windows must be arranged within the same terminal.

However, each Window derives its own geometry from that shared terminal space:

```text
Terminal
   │
   ├── Window A → geometry A
   ├── Window B → geometry B
   └── Window C → geometry C
```

Thus, terminal size is editor-wide while window geometry is Window-specific.

This distinction prevents global terminal information from being confused with the local layout of an individual Window.

## Scope Determines Command Behavior

Once feature ownership is established, commands can be classified by the state they modify.

A Window-scoped command might change:

```text
Cursor
Viewport
```

A Buffer-scoped command might change:

```text
Document
Dirty state
Undo history
```

An editor-wide command might change:

```text
Active Tab
Active Window
Configuration
```

The Active Buffer is absent from this list because it is not a stored field; it is derived from the Active Window. Commands that change the editing target therefore alter the editor's navigational selection — the Active Tab or the Active Window — rather than a separate buffer pointer.

The command therefore does not need a separate implementation for every Buffer.

Instead, it operates on the appropriate object in the current context.

Conceptually:

```text
Command
   │
   ├── Window state
   │
   ├── Buffer state
   │
   └── Editor-wide state
```

The scope determines the target.

## Rendering Follows the Same Scope Model

The Renderer combines state from several scopes when producing a frame.

For example:

```text
Editor-wide
    │
    ├── Terminal dimensions
    │
    ▼
Window
    │
    ├── Geometry
    ├── Viewport
    ├── Cursor
    │
    ▼
Buffer
    │
    ├── Document
    └── Derived state
```

The Renderer reads these relationships to determine what should appear on screen.

It does not need to move any of that state into a single object.

This is another reason the multi-buffer architecture does not require a fundamentally different rendering mechanism. The Renderer simply resolves the state associated with each Window and Buffer while composing the frame.

## The Scope Boundary Is the Important Part

The purpose of this model is not to create more objects for their own sake.

The purpose is to preserve meaningful ownership boundaries.

A useful mental model is:

```text
Editor-wide
    │
    ├── applies to the editor as a whole
    │
    └── shared by multiple contexts

Buffer
    │
    ├── describes one editing context
    │
    └── survives Window switching

Window
    │
    ├── describes one visual context
    │
    └── may reference any Buffer
```

Once these boundaries are clear, existing features can be integrated without duplicating their implementations or introducing hidden global state.

The multi-buffer architecture therefore does not replace the earlier feature architecture.

It **gives each feature a precise scope within a larger editor state model**.

That is the key integration step: multiple Buffers add multiplicity, but they should not destroy the ownership boundaries that already make the editor understandable.

## Summary

Existing features do not need redesign; each simply needs a precise scope. Document, dirty state, undo history, search, and syntax analysis are buffer-scoped; cursor, viewport, and geometry are window-scoped; configuration and terminal size remain editor-wide. Commands and rendering then resolve their targets through these scopes instead of through global state.

The next section examines what this additional retained state costs: **Section 34.14, Complexity and Resource Costs**.
