# 19.3 Rendering the Document Area

The document area is the main part of the editor's screen. It displays the visible portion of the Document inside the region that remains after the Status Bar and Message Bar have been accounted for.

The Renderer is responsible for taking the visible document lines and placing their characters into the appropriate Screen Buffer cells.

The basic transformation is:

```text
Document
   │
   │ visible lines
   ▼
Renderer
   │
   │ screen cells
   ▼
Screen Buffer
```

## The Document Area

The terminal screen is divided into logical regions:

```text
+----------------------------------+
|                                  |
|        Document Area             |
|                                  |
|                                  |
|                                  |
+----------------------------------+
|          Status Bar              |
+----------------------------------+
|          Message Bar             |
+----------------------------------+
```

The document area occupies the rows available for displaying text.

The final rows are reserved for the Status Bar and Message Bar. Therefore, the number of rows available for document content is smaller than the total terminal height.

For example, if the terminal has 24 rows and the editor reserves two rows for its fixed interface:

```text
Terminal rows:       24
Status Bar:           1
Message Bar:          1
--------------------------------
Document rows:       22
```

The Renderer uses this available region when determining how many document lines can be placed into the current frame.

The exact viewport calculations will be developed further in Chapter 20.

## Rendering a Document Line

Suppose the visible portion of the Document contains:

```text
Hello
World
Terminal editor
```

The Renderer processes each visible line and copies its characters into the corresponding screen cells.

Conceptually:

```text
Document line
     │
     ▼
"Hello"
     │
     ▼
+---+---+---+---+---+
| H | e | l | l | o |
+---+---+---+---+---+
     │
     ▼
Screen Buffer
```

Each character occupies a screen cell.

The Screen Buffer therefore provides a direct representation of the visible text:

```text
Screen row 0 → H e l l o
Screen row 1 → W o r l d
Screen row 2 → T e r m i n a l ...
```

The Renderer performs this operation for each visible document line.

## Mapping Lines to Screen Rows

A document has its own row numbering, while the Screen Buffer has screen row numbering.

If the viewport begins at document row 10, then:

```text
Document row 10 → Screen row 0
Document row 11 → Screen row 1
Document row 12 → Screen row 2
```

The Renderer therefore uses the Viewport when determining where each visible line belongs.

Conceptually:

```text
Document
────────────────────────────
Row 10  ───────────────┐
Row 11  ───────────────┤
Row 12  ───────────────┤
Row 13  ───────────────┤
                       │
                       ▼
                    Renderer
                       │
                       ▼
Screen Buffer
────────────────────────────
Row 0   ← Document 10
Row 1   ← Document 11
Row 2   ← Document 12
Row 3   ← Document 13
```

This mapping is the foundation of scrolling, but the Renderer itself does not decide when the viewport should move. It simply uses the current Viewport state.

## Rendering Short Lines

A document line does not necessarily occupy the entire screen width.

For example:

```text
Hello
```

might be rendered into a 20-column screen region as:

```text
H e l l o
```

followed by empty screen cells.

Conceptually:

```text
+---+---+---+---+---+---+---+---+---+---+
| H | e | l | l | o |   |   |   |   |   |
+---+---+---+---+---+---+---+---+---+---+
```

The unused cells must remain blank.

This is important because the Screen Buffer represents the complete frame, not merely the characters that happen to exist in the Document.

If a previous frame contained more characters on that row, leaving old cells populated could cause stale text to remain visible.

The Renderer therefore treats the entire screen region as part of the frame it is constructing.

## Rendering Empty Lines

Empty lines are valid parts of a text document.

For example:

```text
Hello

World
```

contains an empty line between `Hello` and `World`.

The Renderer must preserve that structure visually:

```text
Hello

World
```

The empty document line corresponds to a screen row containing blank cells.

This is another reason the Renderer operates on document rows rather than simply outputting a sequence of characters. A line is a structural unit of the Document, even when it contains no characters.

## Rendering an Empty Document

An empty Document presents another simple case.

If the document contains no text, the document area contains only blank screen cells:

```text
+----------------------------------+
|                                  |
|                                  |
|                                  |
|                                  |
+----------------------------------+
| Status Bar                       |
+----------------------------------+
| Message Bar                      |
+----------------------------------+
```

There is no document text to copy into the Screen Buffer.

The Renderer still constructs the complete frame, however. The absence of document content is itself a valid state that must be represented correctly.

## Lines Shorter Than the Screen Width

The Renderer must also distinguish between the length of a document line and the width of the terminal.

Consider a 40-column document area containing:

```text
Terminal
```

The line contains only eight characters, but the screen row contains forty cells.

The Renderer places the eight characters into the beginning of the row and leaves the remaining cells blank.

```text
Document:
Terminal

Screen:
Terminal····························
```

Here the dots represent empty screen cells.

The document therefore does not need to be padded with spaces in memory merely to match the terminal width.

The padding is a property of the visual representation, not the Document.

## Lines Longer Than the Screen Width

The opposite situation occurs when a document line is wider than the available screen area.

For example:

```text
This is a very long line of text that extends beyond the terminal width.
```

The Renderer cannot place the entire line into the visible row if the screen has fewer columns.

Only the portion that belongs to the current visible horizontal region can be rendered.

Conceptually:

```text
Document line
───────────────────────────────────────────────
This is a very long line of text...

                 │
                 │ horizontal viewport
                 ▼

Screen
────────────────────────
long line of text...
```

The detailed rules for horizontal offsets and horizontal scrolling are deferred to Chapter 20.

For Chapter 19, the important principle is:

> **The Renderer places only the visible portion of a document line into the Screen Buffer.**

## The Renderer Does Not Change the Document

Rendering a line is a read operation.

If the Document contains:

```text
Hello
```

the Renderer does not create a modified version such as:

```text
Hello··········
```

to fit the screen.

Instead, it reads the existing text and writes its visual representation into the Screen Buffer.

This maintains the separation between the logical document and its presentation:

```text
Document
"Hello"
   │
   │ read
   ▼
Renderer
   │
   │ write
   ▼
Screen Buffer
"H e l l o · · ·"
```

The Document remains unchanged.

## Rendering Boundaries

The Renderer must respect the boundaries of the document area.

Document content must not overwrite the rows reserved for the Status Bar or Message Bar.

For example:

```text
+----------------------------------+
| Document row 0                   |
| Document row 1                   |
| Document row 2                   |
| Document row 3                   |
+----------------------------------+
| Status Bar                       |
+----------------------------------+
| Message Bar                      |
+----------------------------------+
```

Even if more document content exists, the Renderer stops drawing document rows when it reaches the end of the document area.

This keeps the different visual regions independent.

## Document Rendering as a Pure Transformation

At this stage, we can view document rendering as a relatively simple transformation:

```text
Visible Document
       │
       │
       ▼
+------------------+
|     Renderer     |
+------------------+
       │
       ▼
Document Area
in Screen Buffer
```

The Renderer reads:

- the Document,
- the Viewport,
- the Screen Size,

and produces the corresponding document area of the Screen Buffer.

## Engineering Insight

The most important design decision here is to keep the Document representation independent of the terminal dimensions.

A line containing five characters remains a five-character line whether the terminal is 20 columns wide or 200 columns wide.

The Renderer adapts that logical representation to the current screen:

```text
               Same Document
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   20-column terminal     80-column terminal
          │                     │
          ▼                     ▼
   Different visual      Different visual
   representation        representation
```

This separation allows the same Document to be rendered on different terminal sizes without changing the underlying text.

The Document describes the content. The Renderer determines how much of that content can currently be presented.

## Summary

Rendering the document area means copying the visible portion of each document line into the corresponding Screen Buffer cells. The Renderer maps document rows to screen rows through the Viewport, leaves unused cells blank, preserves empty lines and empty documents, and clips content to the document area's boundaries without ever modifying the Document itself.

This keeps the logical Document independent of the terminal dimensions, so the same text can be presented on different terminal sizes.

Now that the document area is established, the next section examines the coordinate translation that makes this mapping precise: how a logical document position becomes a specific screen position.

Continue to **Section 19.4, Translating Document Coordinates to Screen Coordinates**.
