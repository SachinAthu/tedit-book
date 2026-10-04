# 42.8 Mouse Coordinates and Editor Coordinates

A mouse event gives the editor a position on the terminal, but that position is not automatically a position in the document.

This distinction becomes important as soon as the terminal contains multiple visual regions:

```text
+----------------------------------+
| Document Area                    |
|                                  |
|                                  |
|                                  |
+----------------------------------+
| Status Bar                       |
+----------------------------------+
| Message Bar                      |
+----------------------------------+
```

A mouse click at terminal row 8, column 20 does not by itself tell the editor which document line and column the user intended.

The editor must translate the terminal coordinate through its existing layout and viewport model.

## Multiple Coordinate Spaces

Mouse interaction introduces a chain of coordinate spaces:

```text
Terminal Coordinates
        │
        ▼
Window Coordinates
        │
        ▼
Viewport Coordinates
        │
        ▼
Document Coordinates
```

Each space answers a different question.

**Terminal coordinates** identify a position on the complete terminal display.

**Window coordinates** identify a position relative to a particular editor window.

**Viewport coordinates** identify a position within the visible portion of that window.

**Document coordinates** identify a logical position within the underlying document.

These spaces should not be treated as interchangeable.

## Terminal Coordinates

The terminal provides coordinates relative to the terminal screen.

For example:

```text
row    = 12
column = 35
```

The origin is associated with the terminal display rather than with a document.

The terminal may contain:

- document windows,
- status bars,
- message bars,
- borders,
- or other interface elements.

Therefore, the coordinate must first be interpreted against the editor's layout.

## Determining the Target Window

With a single document window, the mapping can be relatively simple.

With multiple windows, the editor must determine which window contains the mouse position.

Consider:

```text
+-------------------------------+
| Window A       | Window B     |
|                |              |
|                |              |
|                |              |
|                |              |
+-------------------------------+
| Status Bar                    |
+-------------------------------+
```

A terminal coordinate can therefore be processed as:

```text
Terminal Coordinate
        │
        ▼
Window Hit Test
        │
        ▼
Target Window
```

The **window hit test** is a layout operation. It determines which visual region owns the coordinate.

If the coordinate lies outside every editable window, it should not be converted into a document position.

For example:

```text
Status Bar
     ↑
not a document coordinate
```

This prevents clicks on interface elements from accidentally moving the cursor.

## Window Coordinates

Once the target window is known, the terminal coordinate can be converted into coordinates relative to that window.

Suppose a window begins at:

```text
terminal row    = 5
terminal column = 10
```

and the mouse event occurs at:

```text
terminal row    = 8
terminal column = 20
```

The relative position is conceptually:

```text
window row    = 8  - 5  = 3
window column = 20 - 10 = 10
```

The exact coordinate convention depends on whether borders, padding, or other layout elements are present.

The important principle is that the conversion must use the **actual window geometry** rather than assuming that every terminal coordinate is a document coordinate.

## Viewport Coordinates

The visible document region is only a portion of the document.

For example:

```text
Document

line 0
line 1
line 2
line 3
line 4
line 5
line 6
line 7
line 8
...
```

If the viewport begins at line 4:

```text
Viewport

screen row 0 → document line 4
screen row 1 → document line 5
screen row 2 → document line 6
screen row 3 → document line 7
...
```

A mouse position within the viewport therefore has to be combined with the viewport's offset.

Conceptually:

```text
Viewport Row
     +
Row Offset
     ↓
Document Row
```

The same applies horizontally when horizontal scrolling is supported:

```text
Viewport Column
      +
Column Offset
      ↓
Document Column
```

This is not a new coordinate system invented specifically for mouse support. It is the existing viewport mapping applied to a new input source.

## Mapping to a Document Position

The complete conceptual transformation is:

```text
Terminal Coordinate
        │
        ▼
Window Coordinate
        │
        ▼
Viewport Coordinate
        │
        ▼
Viewport Offset
        │
        ▼
Document Coordinate
```

For a simple single-window example:

```text
document_row =
    viewport_row + row_offset

document_column =
    viewport_column + column_offset
```

The exact calculation must account for the editor's layout and coordinate conventions.

The key point is that the mouse does not bypass the existing cursor and viewport model.

## Document Coordinates Are Logical Coordinates

The resulting document position must use the same coordinate semantics already established for the Cursor.

The mouse parser should not invent a new interpretation of columns.

For example, a document column is not automatically:

```text
UTF-8 byte offset
```

nor is it necessarily:

```text
terminal screen column
```

The editor's existing cursor model defines what a logical document column means.

This is important because visual width and logical position can differ.

Characters such as TAB may occupy multiple terminal columns while representing a single logical character position.

Therefore:

```text
Terminal Column
       ≠
Document Column
```

The Renderer is responsible for visual expansion, while the Cursor and Document operate in their established logical coordinate space.

## Tabs Demonstrate the Difference

Consider:

```text
"    int"
```

The visible indentation may occupy several terminal columns even though the underlying document contains a TAB character.

A mouse click is expressed in visual terminal coordinates.

The editor must therefore determine which logical document position corresponds to that visual position.

Conceptually:

```text
Mouse X
  │
  ▼
Rendered line geometry
  │
  ▼
Logical document column
```

This is a renderer/layout problem rather than a simple arithmetic subtraction.

The same principle applies to any character whose displayed width differs from its logical representation.

## Clicking Between Characters

A mouse click can also fall somewhere within the visual area occupied by a character.

For example:

```text
A B C D E
^
```

The terminal gives a screen coordinate, but the editor must decide which logical position the coordinate represents.

Possible interpretations include:

```text
before character
after character
nearest character
```

The exact policy is an editor-design decision.

The important architectural point is that the conversion from visual position to logical position belongs at the boundary between terminal/layout coordinates and editor coordinates.

The Document itself should not need to understand terminal coordinates.

## Window Boundaries

A coordinate can also fall on a window boundary.

For example:

```text
+-------------+-------------+
|             |             |
|   Window A  |   Window B  |
|             |             |
+-------------+-------------+
```

If a border occupies a terminal column, that column does not necessarily correspond to document content.

The coordinate-mapping stage must therefore understand the window layout.

This is another reason not to place mouse-coordinate handling directly inside the Document or Cursor components.

## Status Bar and Message Bar

The same reasoning applies to non-document interface regions.

Consider:

```text
+-----------------------------+
| Document                    |
|                             |
+-----------------------------+
| Status Bar                  |
+-----------------------------+
| Message Bar                 |
+-----------------------------+
```

A click in the status bar might eventually trigger a UI action.

A click in the message bar might have another meaning.

A click in the document area may move the cursor.

Therefore, the first question should not be:

> Which document position did the user click?

It should be:

> Which visual region did the user click?

Only a coordinate belonging to an editable document region should proceed through document-coordinate mapping.

## Coordinate Conversion as a Boundary

This suggests a useful separation:

```text
Terminal Input
      │
      ▼
Mouse Event
      │
      ▼
Coordinate Mapping
      │
      ├── UI region
      ├── Window
      ├── Viewport
      └── Document position
              │
              ▼
        Editor Command
```

The coordinate-mapping stage translates **where the user interacted** into a position meaningful to the editor.

The resulting command might then be:

```text
MOVE_CURSOR_TO(row, column)
```

The Command Engine remains responsible for executing the command and maintaining the appropriate state invariants.

## Coordinate Mapping and State Ownership

The coordinate conversion process must also respect the existing ownership model.

The layout and viewport information belongs to the editor's existing state and rendering architecture.

The Document owns document content.

The Cursor owns the logical editing position.

The mouse event does not become a second source of truth for cursor location.

Instead:

```text
Mouse
  │
  ▼
Requested Position
  │
  ▼
Cursor
  │
  ▼
Validated Editor State
```

The cursor remains the authoritative representation of the current editing position.

This follows the established architectural rule that each piece of state should have a single owner.

## Coordinate Mapping and Rendering

There is an important relationship between rendering and mouse input.

The Renderer determines how logical document positions become visual positions:

```text
Document Coordinate
       │
       ▼
Viewport
       │
       ▼
Window Layout
       │
       ▼
Terminal Coordinate
```

Mouse interaction needs the reverse operation:

```text
Terminal Coordinate
       │
       ▼
Window Layout
       │
       ▼
Viewport
       │
       ▼
Document Coordinate
```

Conceptually:

```text
        Rendering
Document ─────────────► Terminal


        Input Mapping
Document ◄───────────── Terminal
```

This does **not** mean the renderer should become responsible for input handling.

It means both directions must agree on the same geometry.

If rendering and input mapping use different coordinate rules, a click will not correspond to the text that the user sees.

## The Deeper Architectural Lesson

Mouse support therefore reveals an important property of interactive terminal applications:

> **Visual coordinates and logical coordinates are different domains that require an explicit mapping between them.**

The terminal knows about rows and columns.

The window system knows about regions and layout.

The viewport knows which portion of the document is visible.

The Document knows about logical text positions.

The Cursor knows the current editing position.

Each component can therefore retain its existing responsibility:

```text
Terminal
   │
   │ visual coordinates
   ▼
Layout / Viewport Mapping
   │
   │ logical coordinates
   ▼
Cursor / Editor State
```

This keeps mouse support from introducing a second coordinate model into the editor.

The next section returns to the rendering side and examines **how graphical capabilities fit into the existing rendering architecture**.
