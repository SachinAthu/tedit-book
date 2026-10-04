# 42.6 The Kitty Graphics Protocol

The previous section established that terminal images require more than the ordinary character-cell model. A concrete example of this approach is the **Kitty graphics protocol**, a terminal-specific protocol designed to transmit and display graphical data.

Kitty is useful as a case study because it demonstrates the architectural consequences of extending a terminal beyond ordinary text and color.

The important point is not to make the editor depend on Kitty. The purpose is to understand what happens when a terminal provides a richer graphics protocol.

## A Protocol Beyond Character Cells

Ordinary terminal output is fundamentally cell-oriented:

```text
Character
    ↓
Terminal Cell
    ↓
Display
```

A graphics protocol introduces another representation:

```text
Image Data
    ↓
Graphics Protocol
    ↓
Terminal Graphics Object
    ↓
Display
```

The terminal emulator interprets the graphics protocol and manages the resulting graphical content.

The editor therefore still does not draw pixels directly.

The relationship remains:

```text
Editor
   │
   │ protocol data
   ▼
PTY
   │
   ▼
Kitty-compatible Terminal
   │
   ▼
Display
```

Kitty's protocol is therefore an example of a **terminal protocol extension**, not a replacement for the terminal itself.

## Transmitting Image Data

A graphical image contains substantially more information than can be represented by a sequence of ordinary characters.

A conceptual transmission can therefore be viewed as:

```text
Image
  │
  ▼
Encode image data
  │
  ▼
Transmit through terminal output
  │
  ▼
Terminal interprets graphics data
  │
  ▼
Image becomes available for display
```

The protocol must provide a way to distinguish graphical data from ordinary terminal output.

This is fundamentally different from printing:

```text
printf("Hello");
```

where the output naturally belongs to the character stream.

Graphical data requires protocol-level interpretation.

## Image Identity and Placement

A useful property of a graphics protocol is the ability to distinguish the graphical resource from its position on the display.

Conceptually:

```text
Graphics Resource
        │
        │ identity
        ▼
   Image Object
        │
        │ placement
        ▼
 Terminal Display
```

This separation matters because the same graphical resource may be manipulated independently of the terminal's ordinary character-cell contents.

For an editor, that means graphical output cannot always be treated as simply another sequence of characters inside the Screen Buffer.

## The Difference Between Data and Display

Consider an image associated with a document preview.

The editor may conceptually have:

```text
Image Data
    +
Display Position
```

The image data describes **what** should be displayed.

The position describes **where** it should appear.

These are different responsibilities.

A useful abstraction is:

```text
Graphical Object
├── Resource
│     └── image data
│
└── Presentation
      ├── position
      └── display properties
```

The exact representation belongs to a future implementation. At the architectural level, separating resource data from placement prevents graphical output from being confused with document text.

## Kitty Is Terminal-Specific

The Kitty graphics protocol is not a universal terminal standard.

This distinction is essential:

```text
Terminal graphics
       │
       ├── general concept
       │
       └── Kitty graphics protocol
                 │
                 └── one concrete implementation
```

A terminal application may therefore support Kitty graphics without that implying that every terminal can display the same graphical output.

This reinforces the capability model introduced earlier:

```text
Desired graphical feature
          │
          ▼
Terminal capability
          │
      ┌───┴───┐
      ▼       ▼
   Kitty    unsupported
      │       │
      ▼       ▼
   Render   fallback
```

The application should never make ordinary editing dependent on the presence of the Kitty protocol.

## Kitty and the Terminal Interface

The existing architecture gives us a natural boundary for such a protocol.

The System Architecture defines the Terminal Interface as the component responsible for terminal-specific operations, including terminal dimensions, ANSI output, cursor visibility, and terminal state management.

A future graphics-capable implementation could therefore extend that boundary conceptually:

```text
Terminal Interface
├── Text Output
├── Color Output
├── Cursor Control
├── Terminal State
└── Graphics Output
```

The Core should not contain Kitty protocol sequences.

For example, this would be an undesirable dependency:

```text
Command Engine
      │
      ▼
Kitty Graphics Sequence
```

Instead:

```text
Command / Editor State
          │
          ▼
       Renderer
          │
          ▼
 Terminal Interface
          │
          ▼
 Kitty Graphics Protocol
```

This follows the existing architectural rule that terminal protocol details should remain below the higher-level editor logic.

## Graphics and the Screen Buffer

The Editor Screen Buffer represents the completed visual frame for ordinary terminal-cell rendering. ADR-017 deliberately uses this representation to separate frame construction from terminal output and to keep rendering immediate-mode and predictable.

Kitty graphics introduce an important boundary to that model.

A graphical object cannot simply be represented as:

```text
ScreenCell
```

because its graphical data exists outside the normal cell representation.

Therefore, an extended renderer could conceptually have:

```text
Editor State
      │
      ▼
   Renderer
    /     \
   /       \
Cells     Graphics
  │           │
  ▼           ▼
Screen      Graphics
Buffer      Commands
  │           │
  └─────┬─────┘
        ▼
Terminal Interface
```

The baseline Screen Buffer can remain responsible for character-cell output.

Graphics can follow a separate path.

## The Problem of Full Redraw

This distinction becomes particularly important because our editor deliberately chose **full redraw**.

ADR-002 establishes full redraw as the rendering strategy because redrawing the entire screen avoids the complexity of tracking incremental changes.

For ordinary cells, the model is straightforward:

```text
Previous frame
      ↓
Discard
      ↓
Construct new frame
      ↓
Render
```

Graphical objects can complicate this assumption.

A graphical resource may have a lifetime that extends beyond one ordinary frame. Therefore, an extended renderer may need to distinguish between:

```text
Frame content
```

and:

```text
Persistent graphical resources
```

This means that adding graphics could require additional state at the platform or rendering boundary even though the ordinary editor state remains unchanged.

## Why This Does Not Justify a New Core Model

It might be tempting to redesign the editor around a general graphical scene:

```text
Scene
├── Text
├── Images
├── Shapes
└── Other graphics
```

That would be inappropriate for the editor developed in this book.

The architecture intentionally centers the application around editable text and keeps platform-specific behavior separate. The Core Layer contains the Editor State, Document, Cursor, and Viewport, while terminal interaction belongs to the Platform Layer.

Kitty therefore demonstrates a useful architectural principle:

> **A more capable output device does not automatically require a more general application core.**

The editor should only introduce new core abstractions when the application's requirements actually demand them.

## Kitty as an Optional Capability

For our editor, Kitty graphics would be an optional terminal capability.

The decision path would therefore be:

```text
Editor wants graphical output
            │
            ▼
Is graphical output required?
       /             \
     no               yes
     │                 │
skip graphics     Check capability
                       │
                 ┌─────┴─────┐
                 ▼           ▼
              supported   unsupported
                 │           │
                 ▼           ▼
             use Kitty     fallback
```

The fallback is an architectural requirement, not merely a user-interface convenience.

If the terminal does not provide the required graphics capability, the editor must continue to support its fundamental text-editing operations.

## Kitty and Portability

Kitty also demonstrates why terminal portability becomes increasingly difficult as applications use advanced features.

The baseline editor can rely on a relatively small set of terminal operations.

An editor that depends on Kitty graphics introduces another environmental requirement:

```text
Editor
  │
  ├── POSIX environment
  ├── terminal input
  ├── terminal output
  └── optional graphics protocol
             │
             ▼
       Kitty-compatible
          terminal
```

The more optional capabilities an editor requires, the larger the environmental assumption becomes.

This does not make such features undesirable. It means that they should be explicitly classified as optional capabilities rather than silently becoming requirements.

## The Architectural Lesson

Kitty graphics provide a concrete example of the distinction established throughout this chapter:

```text
Core Meaning
     │
     ▼
Presentation
     │
     ▼
Terminal Capability
     │
     ▼
Terminal Protocol
```

The Core should describe the application's behavior.

The Renderer should determine the visual representation.

The Terminal Interface should handle terminal-specific protocols.

The terminal emulator should interpret those protocols and produce the actual display.

For the editor developed in this book, Kitty graphics therefore remain a **future platform capability**, not part of the baseline implementation.

The value of studying the protocol is architectural: it demonstrates how an apparently simple feature such as "display an image" can introduce capability detection, graphical resources, persistent terminal state, protocol-specific behavior, and additional rendering concerns.

## Summary

The Kitty graphics protocol is a terminal-specific example of how graphical data can be transmitted and displayed beyond ordinary cell output.

The protocol distinguishes the graphical resource from its placement, which means graphical output cannot always be treated as another sequence of characters inside the Screen Buffer.

For the editor developed in this book, Kitty graphics remain a future platform capability. Their value is architectural: they demonstrate how "display an image" introduces capability detection, persistent terminal state, and protocol-specific behavior, all of which belong behind the Terminal Interface boundary.

In the next section, we will turn from graphical output to the other major terminal extension: mouse input.
