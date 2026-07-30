# ADR-001

Programming Language

Decision:

C

Reason:

Lowest abstraction.

---

# ADR-002

Rendering Strategy

Decision:

Full redraw

Reason:

Simpler implementation. Redrawing the entire screen on every frame avoids the complexity of tracking incremental changes and keeps the renderer a pure function of editor state.

First referenced: Chapter 4, Section 4.5 (Clearing the Screen)
