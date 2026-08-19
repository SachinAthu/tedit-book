# 20.4 Horizontal Scrolling

Vertical scrolling determines which **rows** of the document are visible. Horizontal scrolling performs the same job for **columns**.

A line may be longer than the terminal's available width. For example, suppose the terminal can display only 20 columns:

```text
The quick brown fox
```

The complete document line might actually be:

```text
The quick brown fox jumps over the lazy dog
```

Only part of the line can be displayed at once.

The Viewport therefore uses `viewport_column` to determine which portion of a long line is currently visible.

## Scroll Right

Suppose the cursor moves beyond the right edge of the visible area:

```text
viewport_column = 0

The quick brown fox
                   ↑
                 cursor
```

If the cursor moves further right, it can no longer be displayed within the terminal. The editor increases `viewport_column`:

```text
viewport_column = 5

uick brown fox jumps
                 ↑
               cursor
```

The document has not changed. The viewport has simply shifted horizontally to expose a later portion of the line.

## Scroll Left

The same process occurs when the cursor moves back toward the beginning of the line.

Suppose the viewport is currently:

```text
viewport_column = 5

uick brown fox jumps
       ↑
     cursor
```

If the cursor moves to a column before the current `viewport_column`, the editor decreases the offset:

```text
viewport_column = 0

The quick brown fox
    ↑
  cursor
```

The viewport therefore maintains the same basic invariant as vertical scrolling:

> **The cursor must remain within the visible horizontal region.**

The two directions can be represented independently:

```text
             Viewport
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
  viewport_row        viewport_column
        │                 │
        ▼                 ▼
   Vertical          Horizontal
   scrolling         scrolling
```

This gives the editor a two-dimensional viewport over the document. The `viewport_row` determines which document rows are visible, while `viewport_column` determines which columns of those rows are visible.

Horizontal scrolling is especially important because the editor does not use soft wrapping. A long line remains a single document line; the viewport simply exposes different portions of it as the cursor moves horizontally.

## Summary

Horizontal scrolling mirrors vertical scrolling for columns. Because the editor does not use soft wrapping, long lines remain single document lines and `viewport_column` exposes different portions of them as the cursor moves.

In the next section, we will formalize the invariant that ties both directions together: **keeping the cursor visible** within the viewport.
