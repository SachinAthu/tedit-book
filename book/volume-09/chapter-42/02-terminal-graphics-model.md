# 42.2 The Terminal Graphics Model

The terminal has traditionally been treated as a character display device. An application writes bytes to its output stream, and the terminal emulator interprets those bytes as characters, control sequences, and visual attributes.

For the baseline editor, this model is sufficient:

```text
Application
     │
     │ bytes
     ▼
Terminal Emulator
     │
     ▼
Character Grid
```

The terminal emulator maintains the actual visual representation. The editor does not own the physical screen and does not directly manipulate pixels.

Advanced terminal capabilities extend this model without fundamentally changing that relationship.

## The Terminal as a Protocol Endpoint

A terminal graphics feature is still communicated through the terminal connection.

The editor produces a protocol representation:

```text
Editor
   │
   │ terminal protocol
   ▼
PTY
   │
   ▼
Terminal Emulator
   │
   │ interpretation
   ▼
Display
```

The terminal emulator remains responsible for interpreting the protocol and producing the visible result.

This is an important distinction from graphical applications that communicate directly with a windowing system or graphics API.

A terminal editor does not normally receive a framebuffer and draw pixels into it. Instead, it sends information to another program that owns the display surface.

## Two Kinds of Terminal Output

For the purposes of this chapter, terminal output can be divided into two broad categories.

The first is **cell-oriented output**:

```text
Character
Foreground color
Background color
Attributes
Cursor movement
```

This output operates on the terminal's character grid.

The second is **graphics-oriented output**:

```text
Image data
Image placement
Graphics-specific commands
Graphics resources
```

Graphics-oriented protocols may maintain information that does not fit naturally into the ordinary screen-cell model.

This distinction becomes important when considering the architecture of the renderer.

## Cell-Oriented Rendering

The existing editor architecture is based on a screen buffer.

Conceptually:

```text
Screen Buffer
+----+----+----+----+----+
| H  | e  | l  | l  | o  |
+----+----+----+----+----+
| W  | o  | r  | l  | d  |
+----+----+----+----+----+
```

Each visible position corresponds to a cell.

The renderer transforms editor state into this representation and then emits the necessary terminal output.

The terminal subsequently interprets that output and updates its own internal screen representation.

The important ownership boundary is:

```text
Editor owns:
    Screen Buffer

Terminal owns:
    Terminal display state
```

The editor describes what should be displayed. The terminal decides how that description becomes an actual display.

## Graphics-Oriented Rendering

An image introduces another kind of information.

Consider a conceptual image:

```text
+-----------------------+
|                       |
|       IMAGE           |
|                       |
+-----------------------+
```

Its visual content cannot be represented simply as:

```text
one character + one color
```

Even if the image occupies a rectangular region of the terminal, the image itself has its own dimensions and graphical data.

A graphics protocol therefore needs to communicate information beyond ordinary cell contents.

Conceptually:

```text
Image
  │
  ├── graphical data
  ├── dimensions
  └── placement
          │
          ▼
   Graphics Protocol
          │
          ▼
   Terminal Emulator
```

The terminal emulator may then associate that graphical object with a region of the visible display.

## Display Coordinates Still Matter

Graphics do not eliminate the terminal's character grid.

In many terminal graphics systems, graphical content is positioned relative to the terminal's existing coordinate system.

For example:

```text
Terminal
+--------------------------------+
|                                |
|   +----------------------+     |
|   |                      |     |
|   |       Image          |     |
|   |                      |     |
|   +----------------------+     |
|                                |
+--------------------------------+
```

The terminal still has rows and columns, but a graphical object can occupy an area associated with those coordinates.

This creates an important relationship between two concepts:

```text
Character-cell geometry
          +
Graphical-object geometry
```

An advanced terminal application therefore needs to reason about both when it uses graphical capabilities.

## Terminal Graphics Are Not a New Core Representation

It would be tempting to conclude that the editor now needs a completely different internal representation:

```text
Document
├── Text
├── Colors
├── Images
└── Graphics
```

That would be inappropriate for the baseline editor.

The Document represents editable text. Terminal graphics are a presentation capability.

A better separation is:

```text
                    Editor Core
                        │
                 Document State
                        │
                        ▼
                    Renderer
                   /        \
                  /          \
          Cell Rendering   Graphics Rendering
                │                 │
                ▼                 ▼
        Terminal Interface   Graphics Protocol
                  \             /
                   \           /
                    ▼         ▼
                   Terminal
```

The exact implementation may vary in a future editor, but the architectural principle remains:

> **A terminal capability should not become part of the core model merely because the terminal can display it.**

## Protocol-Specific Behavior

Another important property is that terminal graphics are generally **protocol-dependent**.

The editor cannot assume that every terminal understands every graphics command.

Conceptually:

```text
Application
     │
     ├── Standard terminal features
     │
     └── Optional graphics protocol
                 │
                 ▼
          Capability-dependent
                 │
          ┌──────┴──────┐
          ▼             ▼
      Supported      Unsupported
          │             │
       Render        Fallback
```

This is different from ordinary text output, where the basic terminal model is comparatively consistent.

Consequently, advanced graphical features require the editor to know not only **what it wants to display**, but also **whether the current terminal environment can represent that display**.

Capability detection will therefore become an essential part of the design.

## The Architectural Boundary

The existing architecture already provides a natural location for this responsibility.

The Terminal Interface is responsible for terminal-specific operations such as:

- terminal dimensions,
- raw mode,
- ANSI output,
- cursor visibility,
- terminal restoration.

Terminal graphics extend this same platform boundary rather than requiring the Core Layer to understand terminal protocols.

The resulting conceptual separation is:

```text
+-----------------------------+
|         Editor Core         |
|                             |
| Document / Cursor / Viewport|
+--------------+--------------+
               |
               ▼
+-----------------------------+
|          Renderer           |
+--------------+--------------+
               |
               ▼
+-----------------------------+
|      Terminal Interface     |
|                             |
| Text / Colors / Graphics    |
+--------------+--------------+
               |
               ▼
+-----------------------------+
|       Terminal Emulator     |
+-----------------------------+
```

This keeps terminal-specific knowledge at the system boundary.

## Why the Model Matters

Terminal graphics are therefore not simply a collection of additional escape sequences.

They introduce a broader engineering problem:

```text
What should the editor display?
            │
            ▼
How can the renderer represent it?
            │
            ▼
Does the terminal support it?
            │
            ▼
Which protocol should be used?
            │
            ▼
What happens if it is unavailable?
```

The first question belongs primarily to the editor's presentation logic.

The middle questions belong to the platform and terminal interface.

The final question belongs to the capability and fallback strategy.

Keeping these concerns separate allows advanced terminal features to be added without weakening the architecture established for the baseline editor.

The next sections examine individual capabilities, beginning with **true color**, which extends the visual styling of ordinary character cells without requiring a fundamentally different rendering model.
