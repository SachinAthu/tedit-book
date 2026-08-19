# 20.5 Keeping the Cursor Visible

The primary responsibility of scrolling is to ensure that the logical **Cursor** remains within the visible region of the terminal.

After a cursor movement or editing operation, the editor checks whether the cursor is still visible. If it is outside the viewport, the appropriate offset is adjusted.

For vertical scrolling, the editor compares the cursor's document row with the viewport boundaries.

If:

```text
cursor_row < viewport_row
```

the cursor is above the visible region, so the viewport must scroll upward.

If:

```text
cursor_row >= viewport_row + visible_rows
```

the cursor is below the visible region, so the viewport must scroll downward.

Conceptually:

```text
              Cursor position
                     │
                     ▼
              Is cursor visible?
                /           \
              Yes            No
               │              │
               ▼              ▼
          Keep viewport    Adjust offset
                              │
                              ▼
                       Cursor visible
```

The same principle applies horizontally.

If:

```text
cursor_column < viewport_column
```

the cursor is to the left of the visible region, so `viewport_column` must decrease.

If:

```text
cursor_column >= viewport_column + visible_columns
```

the cursor is to the right of the visible region, so `viewport_column` must increase.

The important point is that the viewport does not need to move every time the cursor moves. If the cursor remains within the visible region, the offsets remain unchanged.

For example:

```text
Cursor moves
     │
     ▼
Check viewport boundaries
     │
     ├── Cursor visible ──► Keep current offsets
     │
     └── Cursor outside ──► Adjust offsets
                                  │
                                  ▼
                         Render updated frame
```

This gives us a simple invariant for the editor:

> **After viewport adjustment, the Cursor must be within the visible editing region.**

The Renderer can then rely on this state when constructing the frame. It does not need to decide whether scrolling is necessary; it simply displays the portion of the document identified by the current Viewport.

## Summary

After any cursor movement or editing operation, the editor checks whether the cursor remains within the visible region and adjusts the appropriate offset only when necessary. This establishes the invariant that, after adjustment, the cursor is always visible.

In the next section, we will examine how this check interacts with **cursor movement** and why the Cursor and Viewport remain separate responsibilities.
