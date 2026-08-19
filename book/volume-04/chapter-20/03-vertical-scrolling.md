# 20.3 Vertical Scrolling

Vertical scrolling controls which rows of the document are visible in the terminal.

Suppose the terminal can display 20 rows:

```text
Visible rows:

100 ──────────────
101
102
...
119 ──────────────
```

If the cursor moves to row 120, the current viewport can no longer display it. The editor must increase `viewport_row` so that the cursor becomes visible.

## Scroll Down

Before scrolling:

```text
viewport_row = 100

100
101
...
119   ← bottom of viewport
120   ← cursor
```

The cursor is outside the visible region, so the viewport moves down:

```text
viewport_row = 101

101
102
...
120   ← cursor
```

The document itself has not moved. Only the viewport's starting position has changed.

## Scroll Up

The same principle applies in the opposite direction.

If the cursor moves above the first visible row:

```text
viewport_row = 101

101   ← top of viewport
102
...
120
```

and the cursor moves to row 100, the editor adjusts the viewport:

```text
viewport_row = 100

100   ← cursor
101
102
...
119
```

This reveals an important distinction:

> **Scrolling is not cursor movement.**

The Cursor represents a logical position within the Document. The Viewport changes only when that position falls outside the visible region.

```text
Cursor moves
     │
     ▼
Is cursor visible?
   │       │
  Yes      No
   │       │
   ▼       ▼
Nothing   Adjust
changes   Viewport
```

This separation allows cursor navigation and viewport management to remain independent responsibilities.

## Summary

Scrolling is not cursor movement. The Cursor holds a logical position in the Document, and the Viewport changes only when that position falls outside the visible region.

In the next section, we will apply the same principle horizontally and see how **`viewport_column`** determines which portion of a long line is visible.
