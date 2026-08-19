# 20.10 Design Alternatives

There are several ways an editor could represent and manage scrolling. The appropriate choice depends on the complexity of the editor and the features it needs to support.

## No Viewport

The simplest approach is to always display the document beginning at its first row and column.

```text
Document
    │
    ▼
Terminal
```

This works only for very small documents. Once the cursor moves beyond the terminal's visible area, there is no mechanism for displaying the hidden content.

For an editor intended to navigate arbitrary documents, this approach is insufficient.

## Vertical Offset Only

A slightly more capable design maintains only a vertical offset:

```text
Viewport
└── viewport_row
```

This supports vertical scrolling but cannot handle lines that are wider than the terminal.

A long line would therefore remain inaccessible once the cursor moves beyond the right edge of the screen.

## Line wrapping / soft wrapping

The distinction is:

- **Horizontal scrolling:** keep the document line intact and move the viewport horizontally to reveal different columns.
- **Soft wrapping:** keep the document line intact but display it across multiple terminal rows.

For example, with horizontal scrolling:

```text
Document line:
The quick brown fox jumps over the lazy dog

Terminal:
The quick brown fox
```

Moving right changes the visible portion:

```text
brown fox jumps ove
```

With soft wrapping:

```text
The quick brown fox
jumps over the lazy
dog
```

The underlying Document still contains **one line**. The wrapping exists only for presentation.

### Why this matters architecturally

Soft wrapping changes the relationship between document coordinates and screen coordinates. A single document row can occupy multiple screen rows, so the Viewport can no longer treat:

```text
document row → screen row
```

as a simple one-to-one mapping.

That introduces additional concerns such as:

- calculating wrapped screen lines
- determining the cursor's visual row
- handling long lines efficiently
- keeping vertical scrolling consistent
- mapping screen positions back to document positions

Our editor deliberately uses **horizontal scrolling instead** because it preserves the simple row/column coordinate model established by the Cursor and keeps the Viewport straightforward.

One important nuance: **soft wrapping should be discussed as an alternative to horizontal scrolling, not as another viewport-offset representation.** It changes how long lines are presented rather than simply changing the viewport's horizontal position.

This also aligns with the project requirements, which explicitly exclude soft word wrapping from the baseline editor.

## Vertical and Horizontal Offsets

Our editor uses two offsets:

```text
Viewport
├── viewport_row
└── column_offset
```

`viewport_row` controls vertical scrolling, while `column_offset` controls horizontal scrolling.

This design is simple and maps directly to the terminal's two-dimensional display area. It also fits naturally with the Cursor's row-and-column coordinates.

The main advantages are:

- Simple state representation
- Constant-time viewport adjustments
- Easy-to-understand coordinate calculations
- Independent vertical and horizontal scrolling
- No additional copy of the Document

## More Sophisticated Viewport Models

Larger editors may require more advanced viewport behavior, such as smooth scrolling, scroll margins, multiple views, or independently scrolling windows.

These approaches become useful when an editor supports features such as split windows or multiple documents. They also introduce additional state and coordination that our single-document editor does not currently require.

For the scope of this project, two simple offsets provide the necessary behavior without introducing unnecessary complexity.

> **Engineering Insight:** A simple viewport is sufficient when the editor has a single rectangular editing area. More sophisticated viewport models become justified when the interface itself becomes more complex.

## Summary

Several viewport designs are possible, from displaying no viewport at all to soft wrapping long lines. Two independent offsets fit this editor best because they preserve the simple row-and-column coordinate model and satisfy the project requirements.

In the next section, we will analyze the **complexity and trade-offs** of the chosen model.
