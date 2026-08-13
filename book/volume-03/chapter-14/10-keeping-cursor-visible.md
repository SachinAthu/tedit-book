# 14.10 Keeping the Cursor Visible

A valid logical Cursor is not necessarily a visible Cursor.

The user may move downward until the Cursor reaches the bottom of the visible area:

```text
┌─────────────────────────┐
│ Line 20                 │
│ Line 21                 │
│ Line 22                 │
│ Line 23                 │
│ Line 24                 │
│ Line 25              ^  │
└─────────────────────────┘
```

If the user presses Down again, the Cursor may move to line 26. That position is valid in the Document, but it is no longer visible.

The Viewport must therefore move to follow the Cursor:

```text
Cursor moves
     │
     ▼
Cursor outside visible area?
     │
    Yes
     │
     ▼
Adjust Viewport
     │
     ▼
Render
```

The basic rule is simple:

- If the Cursor moves above the visible region, move the Viewport upward.
- If the Cursor moves below the visible region, move the Viewport downward.
- If the Cursor is already visible, leave the Viewport unchanged.

For example, if the viewport currently begins at row 20 and displays 10 document rows:

```text
viewport_row = 20
visible rows = 20–29
```

If the Cursor moves to row 30:

```text
cursor_row = 30
```

the Viewport must move so that row 30 becomes visible.

After adjusting:

```text
viewport_row = 21
```

the visible range becomes:

```text
21–30
```

and the Cursor is visible again.

## Cursor Visibility Is a Viewport Responsibility

The Cursor does not need to manipulate the terminal when it moves.

Instead, the sequence remains:

```text
Cursor Movement
      │
      ▼
Update Editor State
      │
      ▼
Adjust Viewport
      │
      ▼
Renderer
      │
      ▼
Terminal
```

This keeps responsibilities separate.

The Cursor determines **where the user is**.

The Viewport determines **whether that position is visible**.

The Renderer determines **how that visible state is displayed**.

Horizontal scrolling follows the same principle. If the Cursor moves beyond the visible horizontal range, the horizontal viewport offset is adjusted so that the Cursor returns inside the visible region.

The result is an important invariant for the user interface:

> **After navigation and viewport adjustment are complete, the logical Cursor should always have a corresponding visible screen position.**

This invariant will become especially important when the Renderer and scrolling system are implemented later in the book.
