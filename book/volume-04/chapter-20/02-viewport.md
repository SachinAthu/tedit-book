# 20.2 The Viewport

The terminal cannot display the entire document at once. The **Viewport** represents the portion of the document that is currently visible.

Conceptually:

```text
Document

Line 1
Line 2
Line 3
...
Line 1000
        │
        ▼
   ┌───────────────┐
   │ Visible Area  │
   │               │
   │ Line 240      │
   │ Line 241      │
   │ Line 242      │
   │ ...           │
   │ Line 260      │
   └───────────────┘
```

The Viewport does not contain another copy of the document. Instead, it stores **offsets** that identify where the visible region begins.

For our editor, the important values are:

```text
viewport_row
viewport_column
```

`viewport_row` identifies the first visible document row, while `viewport_column` identifies the first visible **visual/display column** of the rendered line.

The two offsets therefore live in different coordinate spaces. `viewport_row` counts document rows; `viewport_column` counts visual columns — the display coordinates the Renderer produces when it expands the logical line into terminal cells, including TAB expansion (Chapter 19). `viewport_column` is not a UTF-8 byte offset and not a logical Document character index. The two kinds of column coincide only when a line contains no variable-width characters, so that one logical character always occupies exactly one cell.

For example:

```text
viewport_row    = 240
viewport_column = 10
```

means the terminal begins displaying the document at row 240 and visual column 10.

As the user moves through the document, these offsets change when necessary. The Renderer then uses the updated Viewport to determine which portion of the document belongs in the current frame.

The Viewport therefore acts as the boundary between the **logical document** and the **visible terminal region**: vertical in document rows, horizontal in visual columns.

## Summary

The Viewport does not copy the document. It stores two offsets — `viewport_row` and `viewport_column` — that identify where the visible region begins: vertically in document rows, horizontally in visual/display columns, acting as the boundary between the logical document and the visible terminal.

In the next section, we will examine **vertical scrolling** and see how `viewport_row` changes when the cursor moves outside the visible region.
