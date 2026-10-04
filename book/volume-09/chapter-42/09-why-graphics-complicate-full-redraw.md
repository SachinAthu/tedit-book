# 42.9 Why Graphics Complicate Full Redraw

The baseline renderer deliberately uses a simple rendering strategy: construct a complete visual frame and redraw the terminal. The Renderer reads the current Editor State, generates the complete frame, and does not modify the editor state. 03-system-architecture

This works particularly well for ordinary terminal cells because the Screen Buffer represents the visible cell state. A new frame can replace the previous frame without requiring the renderer to understand the history of every individual screen change.

Terminal graphics introduce a complication:

> **Graphical objects may have state that is not represented by the ordinary Screen Buffer.**

This does not invalidate full redraw. It means that the simple model becomes less complete when graphical resources are introduced.

## Ordinary Text Rendering

For ordinary text, the rendering model can be represented as:

```text
Editor State
     │
     ▼
Screen Buffer
     │
     ▼
Terminal Output
     │
     ▼
Terminal Screen
```

Suppose one frame contains:

```text
+----------------------+
| Hello, world!        |
|                      |
| Some source code     |
+----------------------+
```

The next frame might contain:

```text
+----------------------+
| Hello, world!        |
|                      |
| Different text       |
+----------------------+
```

The renderer constructs the new frame and emits the corresponding terminal output.

The previous cell contents do not need to remain as application-level objects.

The current frame is what matters.

## Graphical Objects Are Different

Now consider an image:

```text
+----------------------+
|                      |
|       IMAGE          |
|                      |
+----------------------+
```

The image may involve information that is not naturally represented by:

```text
ScreenCell
```

Instead, the terminal may maintain a graphical resource or graphical placement separately from the ordinary character-cell state.

Conceptually:

```text
Image Resource
      │
      ▼
Terminal-side Graphics State
      │
      ▼
Visible Image
```

The Screen Buffer does not necessarily contain enough information to describe that state.

This creates a second rendering domain:

```text
                    ┌── Screen Cells
Editor State ───────┤
                    └── Graphics Objects
```

The distinction matters because the baseline full-redraw model was designed around the first domain.

## The Stale Graphics Problem

Consider two consecutive frames.

The first frame contains an image:

```text
Frame A

+----------------------+
| Text                 |
|                      |
|      [ IMAGE ]       |
|                      |
+----------------------+
```

The next frame no longer contains it:

```text
Frame B

+----------------------+
| Text                 |
|                      |
|      New text        |
|                      |
+----------------------+
```

For ordinary cells, the renderer can overwrite the previous characters with the new characters.

But if the image exists as a separate terminal-side graphical object, merely writing new character cells does not necessarily mean that the graphical object has been removed.

The conceptual problem is:

```text
Previous graphics
      │
      ▼
Still exists?
      │
      ▼
Must be removed or replaced
```

Therefore, a graphics-aware renderer may need to manage both:

```text
Current cell frame
+
Current graphical state
```

## Graphics Introduce History

This creates an important difference.

The ordinary full-redraw renderer is intentionally close to an immediate-mode model:

```text
Current Editor State
        │
        ▼
Construct Frame
        │
        ▼
Render Frame
```

Graphical resources can introduce retained state:

```text
Create graphical resource
        │
        ▼
Terminal retains resource
        │
        ├── display
        ├── update
        ├── move
        └── remove
```

The renderer may therefore need to know something about the relationship between the current frame and previously submitted graphical objects.

This is one place where an otherwise simple immediate-mode architecture encounters a more stateful external system.

## Redraw Ordering

Graphics also introduce ordering concerns.

Consider:

```text
1. Clear terminal
2. Draw text
3. Draw status bar
4. Draw image
5. Position cursor
```

The order can matter because graphical content and cell content may occupy overlapping visual regions.

A graphics-aware renderer therefore needs an explicit ordering model:

```text
Prepare Frame
      │
      ▼
Clear / establish terminal state
      │
      ▼
Render cell content
      │
      ▼
Render graphical objects
      │
      ▼
Render interface elements
      │
      ▼
Position cursor
```

The exact order depends on the graphics mechanism and the desired visual result.

The important architectural point is that graphics cannot simply be inserted into the existing rendering pipeline without considering their interaction with the terminal's graphical state.

## Scrolling

Scrolling provides another complication.

The editor's Viewport determines which portion of the Document is visible. The existing architecture keeps the Viewport as part of the Editor State and the Renderer uses that state to construct the visible frame. 03-system-architecture

For ordinary text:

```text
Viewport changes
       │
       ▼
Different document lines become visible
       │
       ▼
Construct new frame
```

With graphics, a graphical object may have its own terminal-side position.

Suppose:

```text
Before scrolling:

Document
+----------------------+
| Text                 |
|                      |
|      IMAGE           |
|                      |
+----------------------+
```

After scrolling:

```text
After scrolling:

Document
+----------------------+
| Different text       |
|                      |
|                      |
|                      |
+----------------------+
```

The renderer must determine whether the graphical object is still visible.

If it is no longer part of the visible region, its graphical representation may need to be removed or hidden.

Thus:

```text
Viewport State
      │
      ▼
Visible graphics
      │
      ▼
Graphics update
```

Graphics therefore add another consideration to viewport-driven rendering.

## Resizing

Terminal resizing creates a similar problem.

The Terminal Interface owns the terminal dimensions, while the Renderer uses the current Editor State to generate the visual frame. 03-system-architecture

For ordinary text:

```text
Terminal resize
      │
      ▼
New screen dimensions
      │
      ▼
Viewport recalculation
      │
      ▼
New frame
```

With graphical objects, the renderer may additionally need to reconsider:

- graphical position,
- graphical size,
- visibility,
- clipping,
- and whether previously displayed objects remain valid.

The terminal's graphical state must therefore remain synchronized with the new layout.

## The Synchronization Problem

The deeper issue is synchronization between two representations:

```text
Editor-side state
       │
       ├── Screen Buffer
       │
       └── Graphics state
                │
                ▼
          Terminal state
```

The Screen Buffer describes the cell-based frame.

The graphics representation describes graphical objects.

The terminal ultimately maintains its own display state.

A graphics-aware renderer must ensure that these representations do not contradict one another.

For example:

```text
Editor believes:
    image is absent

Terminal still contains:
    image
```

This produces an incorrect display even though the Screen Buffer itself is correct.

## Does This Mean Full Redraw Is Wrong?

No.

The existence of this complication does **not** invalidate the full-redraw decision.

The baseline architecture was deliberately chosen because full redraw provides a simple and predictable rendering model. The project structure also explicitly requires that the Renderer never modify editor state and that the Document never depend on the Renderer. 05-project-structure

The important distinction is:

```text
Full redraw of cell content
        ≠
Complete reset of every possible terminal resource
```

A future graphics-aware renderer could still use full redraw for ordinary cells while separately managing graphical resources.

For example:

```text
Editor State
     │
     ▼
Renderer
   /     \
  /       \
Cells    Graphics
  │         │
  ▼         ▼
Screen    Graphics
Buffer    Commands
  │         │
  └────┬────┘
       ▼
Terminal Interface
```

The cell-based portion remains immediate-mode.

The graphics portion may require additional resource management.

## Complexity Without Immediate Implementation

This distinction illustrates an important engineering principle.

A system can have a simple baseline architecture while still being designed with awareness of future complexity.

For our editor:

```text
Baseline

Editor State
     ↓
Renderer
     ↓
Screen Buffer
     ↓
Terminal
```

A future graphics-capable architecture could become:

```text
                  ┌── Screen Buffer ──► Terminal
Editor State ─────┤
                  └── Graphics State ─► Graphics Protocol
```

The second design is more complicated because it has more state and more synchronization requirements.

That complexity should not be introduced merely because the terminal is capable of supporting graphics.

## Why This Matters for Our Architecture

The project deliberately favors low coupling, high cohesion, and one-way dependencies. The Renderer reads the Editor State, while the Core remains independent of platform-specific components. 03-system-architecture

Terminal graphics reinforce why those boundaries matter.

If graphics were allowed to leak into the Document or Editor State merely because the terminal can display images, the Core would become coupled to a presentation feature.

Instead:

```text
Document
   │
   │ editable meaning
   ▼
Editor State
   │
   │ presentation
   ▼
Renderer
   │
   ├── Cell Rendering
   │
   └── Optional Graphics
             │
             ▼
      Terminal Interface
```

The additional complexity remains near the rendering and platform boundary.

## Engineering Insight

The baseline full-redraw strategy is simple because the Screen Buffer provides a bounded, cell-based representation of the current frame.

Terminal graphics challenge that simplicity because graphical resources can introduce **state outside the Screen Buffer**.

That leads to a useful distinction:

```text
Baseline rendering:

Current state
     ↓
Current frame
     ↓
Terminal


Graphics-aware rendering:

Current state
     ↓
Current frame + graphics state
     ↓
Terminal
```

The second model is not inherently wrong. It is simply more complex.

For this reason, terminal graphics are a good example of an important architectural rule:

> **Do not add state-management complexity to the Core merely to support an optional presentation capability.**

Our baseline editor can therefore retain its full-redraw architecture while treating terminal graphics as a future extension of the Renderer and Terminal Interface.

The next issue is that advanced terminal capabilities do not only complicate rendering—they also increase the **portability and maintenance cost** of the editor.
