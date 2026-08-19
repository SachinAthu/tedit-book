# 20.8 Viewport Boundaries

The Viewport must remain within the valid boundaries of the Document and the terminal. Its offsets cannot move to arbitrary positions.

At a minimum, both offsets must remain non-negative:

```text
viewport_row >= 0
viewport_column >= 0
```

At the beginning of the document, the vertical offset is therefore:

```text
viewport_row = 0
```

Likewise, when the cursor is at the beginning of a long line:

```text
viewport_column = 0
```

As the cursor moves through the document, the offsets may increase, but they should never expose a region that lies completely beyond the available document content.

For example, if the document contains 100 lines and the terminal can display 20 lines, the viewport cannot meaningfully begin at an arbitrary position such as row 150. Its maximum useful position is determined by the document length and the number of visible rows.

Conceptually:

```text
Document
│
├── Beginning
│     viewport_row = 0
│
├── Visible regions
│     viewport_row increases
│
└── End
      viewport_row stops at the
      final useful position
```

The same principle applies horizontally. The `viewport_column` should be constrained by the length of the relevant content and the available terminal width.

It is useful to distinguish four different boundaries:

| Boundary          | Determines                                        |
| ----------------- | ------------------------------------------------- |
| Document boundary | Where the document begins and ends                |
| Cursor boundary   | Where the Cursor may be positioned                |
| Viewport boundary | Which document region is currently visible        |
| Terminal boundary | How much of that region can actually be displayed |

These boundaries interact, but they represent different concepts.

The Cursor cannot move outside the Document. The Viewport cannot move outside the useful range of the Document. The terminal determines the size of the visible region.

Keeping these constraints separate makes the scrolling logic easier to reason about and helps prevent off-by-one errors when the document or terminal is very small.

## Summary

The Viewport's offsets must remain within the valid ranges imposed by the Document and the terminal. Distinguishing the document, cursor, viewport, and terminal boundaries keeps the scrolling logic easier to reason about.

In the next section, we will examine how the Viewport **interacts with the Renderer** and where scrolling responsibility belongs architecturally.
