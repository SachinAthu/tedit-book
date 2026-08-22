# 20.6 Scrolling and Cursor Movement

Scrolling is closely related to cursor movement, but the two responsibilities remain separate.

The **Cursor** determines the logical position within the Document:

```text
Cursor
    │
    ▼
row = 250
column = 37
```

The **Viewport** determines whether that position can currently be seen:

```text
Viewport
    │
    ├── viewport_row
    └── viewport_column
```

When a cursor movement changes its logical position, the editor can then check whether the new position is still visible.

```text
Cursor Movement
       │
       ▼
Update Cursor
       │
       ▼
Check Visibility
       │
       ▼
Adjust Viewport if necessary
       │
       ▼
Render
```

For example, moving the cursor down normally changes only the cursor's row:

```text
cursor_row += 1
```

If that movement takes the cursor beyond the bottom of the visible region, the viewport must then be adjusted:

```text
if (cursor_row >= viewport_row + visible_rows)
    viewport_row = cursor_row - visible_rows + 1;
```

Likewise, if the cursor moves above the first visible row, the viewport scrolls up so that the cursor becomes the top visible row:

```text
if (cursor_row < viewport_row)
    viewport_row = cursor_row;
```

The same principle applies to horizontal movement, with the logical-to-visual step established in Section 20.5: the editor first obtains the cursor's visual column from the Renderer's mapping, then compares it against `viewport_column` and the visible width.

This separation is important architecturally. The Cursor is responsible for determining whether a requested movement is valid and updating its logical position. The Viewport is responsible for ensuring that position remains visible. Neither component needs to know how the terminal is actually rendered.

Editing operations can also require viewport adjustment. Inserting or deleting text may change the cursor's position or the dimensions of the document around it. After such operations, the editor should re-evaluate cursor visibility before the next frame is rendered.

The general rule is therefore:

> **Whenever an operation can change the logical cursor position, the editor should ensure that the resulting position remains visible.**

This keeps scrolling independent of the particular command that caused the cursor to move. Whether the movement came from an arrow key, text insertion, deletion, or line splitting, the same viewport visibility rule applies.

## Summary

The Cursor determines a logical position, and the Viewport ensures that position remains visible. Neither needs to know how the terminal is rendered, and the same visibility rule applies regardless of which command moved the cursor.

In the next section, we will see how **editing operations** — insertion, deletion, and line splitting — can also move the cursor and therefore trigger the same visibility check.
