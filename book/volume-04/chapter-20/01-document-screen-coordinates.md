# 20.1 Document Coordinates and Screen Coordinates

Scrolling becomes easier to understand when we distinguish between the coordinate systems involved.

**Document coordinates** describe a position within the document:

```text
row    = 250
column = 37
```

These coordinates do not change when the document is scrolled. They count logical positions: one column per Unicode code point (Chapter 14).

**Screen coordinates** describe where that same position appears within the terminal:

```text
screen_row    = 8
screen_column = 15
```

Horizontally, a third coordinate sits between the two: the **visual/display column**. A logical document column counts code points, while a visual column counts terminal cells. The Renderer performs the conversion, for example by expanding a TAB across several cells (Chapter 19). A logical column is therefore not always a visual column.

The **Viewport** connects these coordinate systems through its offsets:

```text
document_row  = viewport_row + screen_row
visual_column = viewport_column + screen_column
```

Vertically, document rows map directly to screen rows. Horizontally, the viewport offset is applied in visual space, and only the result is a screen column:

```text
document_column (logical)
      │
      │ Renderer: tab expansion
      ▼
visual_column (display)
      │
      │ subtract viewport_column
      ▼
screen_column
```

For example, if `viewport_row` is `242`, document row `250` appears at screen row `8`.

```text
Document
    │
    │ row 250
    ▼
Viewport
    │
    │ viewport_row = 242
    ▼
Screen
    │
    │ row 8
    ▼
Terminal
```

The **Cursor** therefore continues to represent a logical position in the Document, while the Viewport determines where that position appears on the screen — with the Renderer bridging the two horizontally.

## Summary

Document coordinates describe a logical position and do not change when the document scrolls. Screen coordinates describe where that position appears within the terminal. Horizontally, the two are connected through **visual columns**: the Renderer maps the logical column to a visual column, and the Viewport then relates that visual column to a screen column through `viewport_column`. Vertically, document rows map directly to screen rows.

In the next section, we will examine the **Viewport** itself and the offsets it stores to identify the currently visible region of the document.
