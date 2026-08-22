# 20.9 Interaction with the Renderer

The Viewport determines **what portion of the Document is visible**; the Renderer determines **how that portion is displayed**.

After cursor movement or editing, the editor first updates its state and adjusts the Viewport if necessary:

```text
Cursor / Document changes
          │
          ▼
   Adjust Viewport
          │
          ▼
    Editor State
          │
          ▼
       Renderer
          │
          ▼
     Screen Buffer
          │
          ▼
       Terminal
```

The Renderer reads the current `viewport_row` and `viewport_column` to determine which document content belongs in the visible frame. It does not decide whether scrolling should occur and does not modify the Viewport.

The coordinate relationship is also one-directional. The Renderer owns the logical-to-visual mapping — including TAB expansion and the configured tab width (Chapter 19) — and the horizontal viewport offset is applied in that visual space. Scrolling therefore operates on the rendered coordinate space without ever computing tab widths or performing the logical-to-visual conversion itself; it consumes the mapping the Renderer already provides.

This separation preserves the responsibility established in the rendering architecture (Chapter 19, ADR-017): the Renderer is a read-only consumer of the Editor State.

The distinction can be summarized as:

> **Scrolling changes the state that defines the visible region. Rendering displays that region.**

As a result, the Renderer does not need to know why the Viewport changed. It simply renders the current state consistently on every frame.

## Summary

Scrolling changes the state that defines the visible region; the Renderer reads that state and displays it. The Viewport is adjusted before rendering, and the Renderer remains a read-only consumer that never modifies it.

In the next section, we will compare several alternative ways of representing and managing scrolling — including no viewport, a vertical offset only, and soft wrapping — and explain why this editor uses two simple offsets.
