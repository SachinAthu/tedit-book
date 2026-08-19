# 20.1 Document Coordinates and Screen Coordinates

Scrolling becomes easier to understand when we distinguish between two coordinate systems.

**Document coordinates** describe a position within the document:

```text
row    = 250
column = 37
```

These coordinates do not change when the document is scrolled.

**Screen coordinates** describe where that same position appears within the terminal:

```text
screen_row    = 8
screen_column = 15
```

The **Viewport** connects these two coordinate systems through its offsets:

```text
document_row = viewport_row + screen_row
document_column = viewport_column + screen_column
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

The **Cursor** therefore continues to represent a logical position in the Document, while the Viewport determines where that position appears on the screen.

## Summary

Document coordinates describe a logical position and do not change when the document scrolls. Screen coordinates describe where that position appears within the terminal, and the Viewport connects the two through its `viewport_row` and `viewport_column`.

In the next section, we will examine the **Viewport** itself and the offsets it stores to identify the currently visible region of the document.
