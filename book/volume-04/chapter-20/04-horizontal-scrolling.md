# 20.4 Horizontal Scrolling

Vertical scrolling determines which **rows** of the document are visible. Horizontal scrolling performs the same job horizontally, but in a different coordinate space: it operates on **visual/display columns** — the columns the Renderer produces when it expands the logical line into terminal cells (Chapter 19) — rather than on the Document's logical columns.

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

## Why Visual Columns Matter

The examples so far contain no TAB characters, so every logical column coincides with its visual column. That is not always true.

Suppose `tab_width = 4` and the document line is:

```text
A<TAB>123456789
```

The Document contains eleven logical characters: `A`, the TAB, and the nine digits (Chapter 14). The Renderer expands the TAB, so the same line occupies more visual columns:

```text
visual:     A···123456789
visual col: 012345678901234567...
```

The TAB is one logical character at logical column `1`, but it spans visual columns `1` through `3`; the `1` begins at visual column `4`. Therefore:

```text
viewport_column = 4
```

means "the viewport begins at visual column 4" — at the digit `1`, which is only the third logical character. It does **not** mean "the viewport begins at the fourth logical character."

Horizontal scrolling must operate in the visual coordinate space, because that is the space in which the terminal displays the line and in which the Renderer measures it:

```text
logical cursor column
      │
      │ Renderer (tab expansion)
      ▼
visual cursor column
      │
      ▼
horizontal scrolling decision
```

tab expansion itself belongs to the Renderer (Chapter 19). The scrolling logic consumes the visual columns the Renderer produces; it never computes tab widths itself.

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

This gives the editor a two-dimensional viewport over the document. The `viewport_row` determines which document rows are visible, while `viewport_column` determines which visual columns of those rows are visible.

Horizontal scrolling is especially important because the editor does not use soft wrapping. A long line remains a single document line; the viewport simply exposes different portions of it as the cursor moves horizontally.

## Summary

Horizontal scrolling mirrors vertical scrolling horizontally, but in visual/display columns rather than logical columns: the Renderer expands the logical line (including TABs) into visual columns, and `viewport_column` then exposes different portions of those visual columns. Because the editor does not use soft wrapping, long lines remain single document lines and the viewport exposes different portions of them as the cursor moves.

In the next section, we will formalize the invariant that ties both directions together: **keeping the cursor visible** within the viewport.
