# 19.7 Rendering the Cursor

The Cursor is the visual indicator that shows the user where the next editing operation will occur.

The Cursor itself is part of the editor's logical state. The Renderer is responsible for taking that logical position and representing it at the appropriate location in the current frame.

The architecture defines the Cursor as maintaining a document position and a screen position, while the Renderer consumes the current Editor State to generate the complete frame.

The relationship is:

```text
Cursor
   │
   │ logical position
   ▼
Renderer
   │
   │ screen position
   ▼
Terminal
```

## The Logical Cursor Position

The Cursor identifies a position within the Document.

For example:

```text
Hello
World
```

If the Cursor is positioned at the fourth column of `World`, its logical position describes that location within the Document:

```text
row    = 1
column = 4
```

This position belongs to the Cursor, not to the Renderer.

The Renderer reads it when constructing the frame.

```text
Cursor
  │
  ├── row
  └── column
       │
       ▼
    Renderer
```

This maintains the same ownership rule used throughout the architecture: the component that owns a piece of state is responsible for changing it, while other components may read it.

## From Logical Position to Screen Position

The logical Cursor position does not necessarily correspond directly to a terminal position.

The Renderer must account for the current Viewport.

For example, suppose:

```text
cursor_row    = 15
cursor_column = 8

viewport_row    = 12
viewport_column = 3
```

The visible screen position becomes:

```text
screen_row    = 15 - 12 = 3
screen_column =  8 -  3 = 5
```

The Cursor therefore appears at:

```text
(row 3, column 5)
```

within the document area.

The important distinction is:

```text
Logical Cursor
(row 15, column 8)
        │
        │ Viewport translation
        ▼
Visual Cursor
(row 3, column 5)
```

The Renderer performs the translation. It does not modify the logical Cursor to make the values match.

## The Cursor Is Not Part of the Document

The Cursor is a position within the Document, not a character stored in it.

Consider:

```text
Hello
World
```

The Document contains only the text.

The Cursor might be represented visually as:

```text
World
   ^
```

but the `^` is not actually inserted into the Document.

Instead, the Renderer uses the Cursor position to determine where the terminal cursor should appear.

```text
Document
    │
    ├── "Hello"
    └── "World"

Cursor
    │
    └── (1, 3)

        │
        ▼

     Renderer
        │
        ▼

Visual frame
    World
       │
       └── terminal cursor
```

This distinction is essential because moving the Cursor must never modify the Document.

## Positioning the Terminal Cursor

Once the Renderer has calculated the screen position, the terminal-facing part of the rendering pipeline can position the actual terminal cursor there.

The conceptual flow is:

```text
Editor State
     │
     ▼
Renderer
     │
     ├── Draw document
     ├── Draw Status Bar
     ├── Draw Message Bar
     │
     └── Determine cursor position
              │
              ▼
        Screen Buffer / Frame
              │
              ▼
        Terminal Interface
              │
              ▼
        Terminal Cursor
```

The Terminal Interface is responsible for interacting with the terminal and writing ANSI escape sequences. The Renderer determines **where** the cursor belongs in the frame; the terminal-facing layer performs the terminal-specific operation needed to display it.

This keeps terminal protocol details out of the Cursor and Document components.

## The Cursor Must Correspond to the Visible Document

The Cursor should normally be visible within the document area.

Suppose the document area can display five rows:

```text
+----------------------------------+
| Line 10                          |
| Line 11                          |
| Line 12                          |
| Line 13                          |
| Line 14                          |
+----------------------------------+
| Status Bar                       |
+----------------------------------+
| Message Bar                      |
+----------------------------------+
```

If the logical Cursor is on document row 12:

```text
document_row = 12
viewport_row = 10
```

then:

```text
screen_row = 12 - 10
           = 2
```

The Cursor appears on the third visible document row.

If the logical Cursor moves to document row 15, the Renderer would calculate:

```text
screen_row = 15 - 10
           = 5
```

which is outside this five-row document area.

The Renderer should not solve this by moving the Cursor.

Instead, the Viewport must be updated so that the Cursor becomes visible.

This establishes an important division of responsibility:

```text
Cursor
  │
  │ logical position
  ▼
Viewport
  │
  │ determines visible region
  ▼
Renderer
  │
  │ calculates screen position
  ▼
Terminal Cursor
```

The rules for changing the Viewport when the Cursor approaches or crosses the visible boundaries belong to Chapter 20.

## Cursor at the End of a Line

A Cursor can also be positioned at the end of a document line.

For example:

```text
Hello
     ^
```

The logical column can be equal to the line's length.

If:

```text
line = "Hello"
length = 5
```

then a Cursor at:

```text
column = 5
```

represents the position immediately after the final character.

The Renderer must represent that logical position correctly rather than assuming that every Cursor position corresponds to an existing character.

This is particularly important for insertion operations.

If the user types a character at this position:

```text
Hello|
```

the new character is inserted at the Cursor position.

The Cursor therefore represents a **position in the editing model**, not simply the location of a character.

## Cursor on an Empty Line

An empty line contains no characters, but the Cursor can still be positioned on it.

For example:

```text
Hello

World
```

The Cursor can be on the empty line:

```text
Hello
|
World
```

The Renderer still needs to provide a valid screen position for it.

The absence of document characters does not mean that the Cursor disappears.

This reinforces the distinction between the Document's content and the Cursor's position within that content.

## Cursor and the Screen Buffer

The Cursor is slightly different from ordinary document characters.

The document characters become Screen Buffer cells:

```text
Document
   │
   ▼
Screen Buffer
```

The Cursor is a position over that rendered content:

```text
Document
   │
   ▼
Screen Buffer
   │
   └── cursor position
```

Conceptually:

```text
+----------------------------------+
| H e l l o                        |
| W o r l d                        |
|                                  |
+----------------------------------+
             ^
             Cursor
```

The Cursor therefore should not be treated as another character in the Document.

It is a property of the visual frame indicating the active editing position.

## Cursor and Fixed Interface Regions

The Cursor belongs to the document area.

It must not be placed on the Status Bar or Message Bar.

For example:

```text
+----------------------------------+
| Document                         |
|        ^ Cursor                  |
|                                  |
+----------------------------------+
| Status Bar                       |
+----------------------------------+
| Message Bar                      |
+----------------------------------+
```

The Renderer must ensure that the calculated Cursor screen position falls within the document region.

This is another reason the document area, Status Bar, and Message Bar must have clearly defined boundaries.

## Rendering the Cursor Does Not Move It

Rendering must remain read-only.

If the Cursor is currently:

```text
row    = 10
column = 15
```

the Renderer calculates where that position appears on screen.

It does not change the Cursor to:

```text
row    = 2
column = 5
```

just because that is the screen position.

The relationship is:

```text
Logical Cursor
(row 10, column 15)
        │
        │ translate
        ▼
Screen Position
(row 2, column 5)
```

The first pair belongs to the editing model.

The second pair belongs to the current frame.

Confusing the two would cause scrolling and resizing to interfere with the logical editing position.

## Terminal Cursor Versus Cursor State

There are therefore two related but distinct concepts:

**Logical Cursor**

```text
(row, column)
```

The position used by editing operations.

**Terminal Cursor**

```text
(screen_row, screen_column)
```

The position at which the terminal displays the cursor.

They are connected by the Renderer:

```text
Logical Cursor
      │
      │ Viewport
      ▼
Screen Position
      │
      ▼
Terminal Cursor
```

The logical Cursor is part of Editor State.

The terminal cursor is part of the visual presentation.

This distinction allows the editor to scroll without changing the user's logical position.

## A Complete Example

Suppose the Document contains:

```text
Line 10
Line 11
Line 12
Line 13
Line 14
Line 15
```

The current state is:

```text
cursor_row    = 13
cursor_column = 4

viewport_row = 11
```

The visible document area contains:

```text
Line 11
Line 12
Line 13
Line 14
Line 15
```

The Renderer calculates:

```text
screen_row = 13 - 11
           = 2
```

and:

```text
screen_column = 4
```

The resulting frame places the terminal Cursor on the third visible document row:

```text
Line 11
Line 12
Line 13
    ^
Line 14
Line 15
```

The logical state remains:

```text
Cursor:
row    = 13
column = 4
```

The screen position is merely its current visual representation.

## Engineering Insight

The Cursor illustrates the central responsibility of the Renderer particularly well:

> **The Renderer does not own the meaning of a position; it translates that position into a visual representation.**

The Cursor's logical position belongs to the editing model.

The Viewport determines which portion of the Document is visible.

The Renderer combines those values to determine where the Cursor appears in the current frame.

The Terminal Interface then handles the terminal-specific mechanism for displaying the cursor. The architecture therefore remains separated into clear responsibilities:

```text
Cursor
  │
  │ logical position
  ▼
Viewport
  │
  │ visible region
  ▼
Renderer
  │
  │ screen position
  ▼
Terminal Interface
  │
  ▼
Terminal Cursor
```

This separation becomes especially important when the terminal is resized or the document is scrolled. The logical Cursor remains stable while its screen position can change from frame to frame.

## Summary

The Cursor is a logical position within the Document, not a character stored in it. The Renderer translates that logical position into a screen position using the Viewport, ensures it falls within the document area, and leaves the actual terminal cursor placement to the terminal-facing layer.

Because the logical Cursor and its screen position are distinct, the Cursor remains stable while its visual position can change from frame to frame.

Now that each visual element is defined, the next section brings them together by describing the order in which the Renderer constructs a frame.

Continue to **Section 19.8, Rendering Order**.
