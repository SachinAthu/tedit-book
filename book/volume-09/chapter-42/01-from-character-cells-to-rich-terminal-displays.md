# 42.1 From Character Cells to Rich Terminal Displays

The terminal interface developed throughout this book is primarily a character-based display. The editor produces text, positioning commands, and visual attributes, and the terminal emulator interprets those bytes to construct a screen from rows and columns of character cells.

That model is powerful enough to build a complete text editor. The editor can display document text, draw a status bar, position the cursor, apply colors, and update the visible screen without ever manipulating pixels directly.

However, modern terminal emulators can provide capabilities that go beyond the traditional character-cell model. Some terminals support true-color output, mouse events, inline images, and terminal-specific graphics protocols. These capabilities allow terminal applications to create interfaces that are visually richer than a sequence of ordinary characters.

This changes the range of what a terminal application can represent.

## The Traditional Terminal Model

The basic rendering model can be represented as:

```text
Editor State
     │
     ▼
Renderer
     │
     ▼
Screen Buffer
     │
     ▼
Terminal Interface
     │
     ▼
Terminal Emulator
     │
     ▼
Screen
```

The Screen Buffer describes the visible character cells. Each cell corresponds to a position on the terminal grid and can contain information such as a character and its visual attributes.

Conceptually:

```text
+-----+-----+-----+-----+-----+
|  H  |  e  |  l  |  l  |  o  |
+-----+-----+-----+-----+-----+
|  W  |  o  |  r  |  l  |  d  |
+-----+-----+-----+-----+-----+
```

The renderer therefore works primarily in terms of **screen coordinates and cells**.

This model is deliberately simple. It gives the editor a predictable relationship between its logical state and its visible representation.

The editor does not need to know how individual pixels are produced. It only needs to determine what should appear at each visible position.

## Beyond Characters

A modern terminal can expose additional capabilities.

The conceptual model becomes:

```text
                         ┌── Character Cells
                         │
Editor ──► Renderer ─────┼── Colors / Attributes
                         │
                         ├── Mouse Interaction
                         │
                         └── Graphics / Images
```

These capabilities do not all have the same nature.

Color still operates naturally within the character-cell model. A character can have a foreground and background color, for example.

Mouse input is different. It is an **input capability** that allows the terminal to send additional events back to the application.

Images are different again. An image is not naturally represented by one character cell. It may occupy an area of the display that corresponds to many cells while carrying information that cannot be represented by those cells alone.

This distinction is important because adding a capability does not necessarily mean extending the Screen Buffer with another kind of value. Some capabilities belong to different parts of the system.

## Character-Based Rendering and Graphics

The difference can be summarized as follows:

```text
Traditional rendering

Editor State
     │
     ▼
Screen Buffer
     │
     ▼
Character Cells
     │
     ▼
Terminal
```

Whereas graphical terminal features may introduce another path:

```text
Editor State
     │
     ▼
Renderer
     │
     ├──────────────► Screen Cells
     │
     └──────────────► Graphics Protocol
                           │
                           ▼
                        Terminal
```

The second path should not automatically become part of the Document or other core editor structures.

The Document represents editable text. The Cursor represents a logical editing position. The Viewport determines which part of that document is visible. These concepts remain meaningful regardless of whether the terminal happens to support additional visual capabilities.

Terminal graphics therefore belong primarily to the **presentation and platform side of the architecture**.

## Why This Matters for Our Editor

Our editor was intentionally designed around a small and well-defined set of responsibilities.

The Core Layer contains the application's essential state:

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

The platform-facing components deal with terminal-specific behavior.

This separation becomes even more valuable when optional terminal capabilities are introduced.

For example, the Core should not need to know whether the user is running the editor inside a terminal that supports a particular graphics protocol.

Instead:

```text
Core
 │
 │ platform-independent state
 ▼
Renderer
 │
 │ presentation decisions
 ▼
Terminal Interface
 │
 │ terminal-specific protocol
 ▼
Terminal Emulator
```

The terminal interface can determine how a particular capability is expressed without forcing the rest of the editor to depend on that terminal's protocol.

## Capability Is Not the Same as Requirement

A terminal may support a feature without the editor being required to use it.

This distinction is fundamental.

For example, suppose a terminal supports inline images. That does not mean that the editor must support inline images.

Similarly:

```text
Terminal supports feature
            ≠
Editor requires feature
```

The baseline editor can remain a text editor even when executed inside a much more capable terminal.

This allows the architecture to remain stable while optional presentation features are introduced later.

## The Terminal as a Capability Surface

It is therefore useful to think of a modern terminal as a **capability surface** rather than a single fixed display mechanism.

At the simplest level, the terminal provides:

```text
Text
Cursor
Attributes
```

More capable terminals may additionally provide:

```text
True Color
Mouse Events
Images
Graphics Protocols
```

These capabilities are not equally standardized or universally available.

That observation leads to an important architectural requirement for advanced terminal applications:

> **The editor must distinguish its core behavior from capabilities provided by the terminal environment.**

The editor should remain correct when an optional capability is unavailable.

This principle will guide the rest of this chapter as we examine color, capability detection, images, mouse input, and terminal graphics protocols.
