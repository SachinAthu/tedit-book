# 42.5 Images in Terminals

Character cells are an effective representation for text, but they are not a general representation of graphical data.

A character cell can describe something like:

```text
+----------------------+
| Character            |
| Foreground Color     |
| Background Color     |
| Attributes           |
+----------------------+
```

An image contains substantially more information:

```text
+----------------------+
| Pixel | Pixel | ...  |
| Pixel | Pixel | ...  |
| Pixel | Pixel | ...  |
|  ...  |  ...  | ...  |
+----------------------+
```

A terminal that supports images therefore needs a mechanism for transmitting graphical data in addition to ordinary character output.

## Why Images Are Different

Consider a terminal displaying:

```text
Hello, world!
```

The editor can represent this as a sequence of cells:

```text
H e l l o ,   w o r l d !
```

Each character occupies a known position in the terminal grid.

An image does not have the same representation.

A photograph, diagram, or icon may contain thousands or millions of pixels. Reducing it to ordinary characters would discard most of its visual information.

Therefore, an image requires a different representation:

```text
Image Data
    │
    ▼
Graphics Protocol
    │
    ▼
Terminal Emulator
    │
    ▼
Displayed Image
```

The terminal emulator receives the graphical information and is responsible for displaying it.

## Image Data and Image Placement

A terminal graphics system generally has to communicate at least two different kinds of information:

```text
Image
├── Data
└── Placement
```

**Image data** describes the graphical content.

**Image placement** describes where that content should appear.

These concerns are related but should not be confused.

For example:

```text
Image Data
    │
    ▼
+-------------------+
|                   |
|      IMAGE        |
|                   |
+-------------------+
          │
          ▼
Position = terminal row/column
```

The same image data could potentially be displayed at different positions.

This distinction becomes important when an editor needs to update or reposition graphical content.

## Images and the Character Grid

Although an image is not itself a character-cell value, terminal graphics can still interact with the terminal's character grid.

Consider:

```text
Terminal Grid

+---+---+---+---+---+---+---+
|   |   |   |   |   |   |   |
+---+---+---+---+---+---+---+
|   |   | +----------------+ |
+---+---+ |                | |
|   |   | |     IMAGE      | |
+---+---+ |                | |
|   |   | +----------------+ |
+---+---+---+---+---+---+---+
```

The image may occupy a region that is described relative to terminal coordinates, while the terminal still maintains its ordinary character grid.

This means an advanced renderer may need to reason about two related but different concepts:

```text
Character-cell geometry
          │
          └── rows and columns

Graphics geometry
          │
          └── graphical objects and placement
```

The exact relationship depends on the graphics protocol being used.

## Images Are Not Part of the Document

An important architectural question is where image data should live.

The answer should not be:

```text
Document
├── Text
└── Images
```

at least not for the baseline editor described by this book.

The Document represents editable text. The fact that a terminal can display images does not change the fundamental representation of the document.

Instead, graphical content should be treated as presentation information or as an optional feature layered around the editor.

Conceptually:

```text
Document
    │
    ▼
Editor State
    │
    ▼
Renderer
   / \
  /   \
Text   Graphics
 │        │
 ▼        ▼
Cells   Graphics Protocol
  \       /
   \     /
    ▼   ▼
   Terminal
```

This preserves the separation between document data and terminal presentation.

## Images and the Renderer

The renderer now has potentially two output domains.

The first is the ordinary cell-based display:

```text
Document
   ↓
Visible Lines
   ↓
Screen Buffer
   ↓
Terminal Cells
```

The second is graphical output:

```text
Graphical Object
       ↓
Graphics Representation
       ↓
Graphics Protocol
       ↓
Terminal
```

This does not necessarily mean that the renderer must become two completely independent systems.

It means that the renderer must recognize that not every visual element can be represented by a Screen Buffer cell.

That distinction is important when designing future extensions.

## Image Lifetime

Graphical output can also introduce a concept that is less significant in ordinary text rendering: **resource lifetime**.

A character printed to a terminal is part of the terminal's current display state.

A graphics protocol may instead allow graphical data to be transmitted, stored, identified, positioned, and subsequently manipulated by the terminal.

Conceptually:

```text
Create Image
     │
     ▼
Terminal-side Resource
     │
     ├── display
     ├── reposition
     ├── update
     └── remove
```

The exact behavior depends on the terminal graphics protocol.

The architectural lesson is more general:

> **Graphical terminal features can introduce state that exists outside the editor's ordinary Screen Buffer.**

This becomes particularly important for full-screen redraw.

## Images and Full-Screen Redraw

Our baseline renderer uses full-screen redraw because it provides a simple and predictable rendering model.

For ordinary text:

```text
Clear
  ↓
Draw visible cells
  ↓
Draw status bar
  ↓
Draw message bar
  ↓
Position cursor
```

Graphical objects complicate this model.

Suppose an image was previously displayed and the next frame no longer contains it:

```text
Previous Frame:
+----------------------+
|       IMAGE          |
+----------------------+

Next Frame:
+----------------------+
|       Text           |
+----------------------+
```

Simply redrawing the new character cells may not be sufficient if the terminal maintains graphical state independently of those cells.

The renderer may need an explicit mechanism for removing or replacing the graphical object.

This creates a new responsibility:

```text
Previous graphical state
          │
          ▼
Determine what changed
          │
          ▼
Remove / update stale graphics
          │
          ▼
Render current graphics
```

This is one of the reasons terminal graphics are more complicated than simply adding another type of screen cell.

## Images and Scrolling

Scrolling creates another problem.

The editor's Viewport determines which part of the document is visible:

```text
Document
   │
   ▼
Viewport
   │
   ▼
Visible document region
```

A graphical object may be associated with a particular visual location.

When the viewport changes:

```text
Before scrolling

Document region A
      ↓
Image at position X


After scrolling

Document region B
      ↓
Image may no longer be visible
```

The editor therefore needs a clear relationship between graphical objects and the viewport if images are ever integrated into the editing interface.

This is not necessary for the baseline editor because the baseline document consists of text rendered through the ordinary cell model.

## Images and Terminal Resizing

Terminal resizing introduces another consideration.

Suppose an image is displayed relative to a particular region of the terminal:

```text
+-----------------------------+
|                             |
|          IMAGE              |
|                             |
+-----------------------------+
```

If the terminal becomes narrower:

```text
+-------------------+
|                   |
|       IMAGE       |
|                   |
+-------------------+
```

The editor may need to reconsider its placement or dimensions.

This differs from ordinary text rendering, where the existing viewport and screen-buffer logic already establish how visible rows and columns are recalculated.

Graphical objects therefore introduce another dimension to layout management.

## Images and Performance

Images can also have a very different cost from ordinary text.

A text frame might contain a relatively small amount of output:

```text
Characters
   ↓
Terminal sequences
```

An image can require substantially more data:

```text
Image
   ↓
Encoded graphical data
   ↓
Terminal output
```

Repeatedly transmitting large images during every full-screen redraw could be unnecessarily expensive.

This creates a tension with the baseline rendering strategy:

```text
Full redraw
     +
Large graphical resources
     ↓
Potentially expensive output
```

A future graphics-aware renderer may therefore need to distinguish between:

- persistent graphical resources,
- graphical placement,
- and ordinary frame content.

This is another example of how an advanced feature can challenge a deliberately simple baseline architecture.

## Protocol Dependence

Images are not a universal terminal primitive in the same way that ordinary characters are.

Different terminal environments may support different image mechanisms, or no image mechanism at all.

Therefore:

```text
Image
  │
  ▼
Supported graphics protocol?
  │
 ┌┴───────────┐
yes           no
 │             │
 ▼             ▼
Display      Fallback
```

The editor should not make ordinary text editing dependent on image support.

If graphical rendering is unavailable, the editor should remain usable.

## Appropriate Use in a Text Editor

The existence of terminal image support does not mean that every text editor needs it.

Possible uses include:

- displaying image previews,
- showing diagrams,
- previewing generated content,
- displaying documentation assets,
- presenting graphical diagnostics,
- or providing richer interface elements.

However, these are optional capabilities.

The core operation of the editor remains:

```text
Edit text
   ↓
Navigate text
   ↓
Save text
```

Images should therefore be treated as an extension rather than a fundamental part of the document-editing model.

## Architectural Conclusion

For the architecture established in this book, terminal images should remain outside the baseline implementation.

If image support were added later, the conceptual boundary would be:

```text
                  Editor Core
                       │
                       ▼
                    Renderer
                   /        \
                  /          \
          Screen Buffer    Graphics
               │               │
               ▼               ▼
        Terminal Interface  Graphics
                              Protocol
                  \             /
                   \           /
                    ▼         ▼
                    Terminal
```

The Screen Buffer continues to represent ordinary cell-based output.

Graphical objects follow a separate presentation path.

The Terminal Interface remains responsible for translating those graphical requests into the protocol understood by the target terminal.

This preserves the central architectural principle of the editor:

> **The Core describes the editor's meaning; the platform layer determines how that meaning is presented by the environment.**

The next section examines **Kitty's graphics protocol** as a concrete example of how a modern terminal can implement this model.
