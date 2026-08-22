# 20.11 Complexity and Trade-offs

The scrolling model is intentionally simple. The Viewport contains only the offsets required to identify the visible region, and adjusting those offsets requires only a small number of coordinate comparisons.

## Time Complexity

Keeping the cursor visible requires checking the cursor against the viewport boundaries:

```text
cursor_row
cursor_visual_column
viewport_row
viewport_column
```

Each check takes constant time, so viewport adjustment is:

```text
O(1)
```

The cost does not depend on the size of the document. Whether the document contains 100 lines or 100,000 lines, determining whether the cursor is outside the current viewport requires the same basic comparisons.

These comparisons operate in the coordinate spaces established in this chapter: rows are logical document rows, and horizontal positions are visual columns. Obtaining the cursor's visual column requires the Renderer's tab expansion, whose cost is part of rendering (Chapter 19), not part of the viewport adjustment itself.

## Space Complexity

The Viewport stores only a constant amount of state:

```text
viewport_row
viewport_column
```

Therefore, the additional space required by the scrolling mechanism is:

```text
O(1)
```

The Viewport does not duplicate document contents or create another representation of the visible text.

## Trade-offs

The simplicity of this model comes with deliberate limitations.

Horizontal scrolling keeps each document line intact, which preserves the straightforward row-and-column coordinate model. However, very long lines can require the user to scroll horizontally rather than seeing the entire line at once.

Soft wrapping could make long lines easier to read, but it would introduce a more complicated relationship between document coordinates and screen coordinates.

For the baseline editor, the simpler offset-based model is preferable:

```text
Simple state
      ↓
Simple calculations
      ↓
O(1) viewport adjustment
      ↓
Predictable behavior
```

This is consistent with the project's broader goal of prioritizing simplicity and educational value over advanced interface behavior.

## Summary

Keeping the cursor visible requires only constant-time coordinate comparisons, and the Viewport stores a constant amount of state, giving O(1) time and space complexity. The simple offset-based model is deliberately preferred for the baseline editor.

In the next section, we will examine the **common scrolling bugs** that arise when these boundaries and rules are handled incorrectly.
