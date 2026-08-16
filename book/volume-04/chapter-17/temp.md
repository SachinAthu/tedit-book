# Chapter 17 — Rendering Theory

# 17.8 Double Buffering

Introduce the problem of constructing a visual result while another result is currently visible.

Conceptually:

```text
Application
    │
    ▼
Back Buffer
    │
    │ complete frame
    ▼
Front Buffer
    │
    ▼
Display
```

Explain what double buffering is intended to achieve:

- prevent partially constructed frames from being displayed
- allow a complete frame to be prepared before presentation
- reduce visible tearing or intermediate states

Then make an important distinction for this project:

### Terminal applications do not have the same hardware-backed front/back buffers as graphical applications.

Our editor will use **application-side screen representation** rather than relying on a graphical display buffer.

That leads directly to Chapter 18.

This distinction is important enough to make explicit so readers don't incorrectly assume that terminal applications have GPU-style framebuffers.

---

# 17.9 Rendering Models in a Terminal

Now bring the theory specifically into the terminal environment.

The terminal ultimately receives a stream of bytes:

```text
Editor
   │
   ▼
ANSI escape sequences + text
   │
   ▼
Terminal emulator
   │
   ▼
Visible screen
```

The renderer therefore does not directly manipulate pixels.

This connects naturally back to Chapters 2–4 without repeating their explanations.

The renderer's job is essentially:

> Convert the editor's logical state into a sequence of terminal operations that produces the desired visual frame.

---

# 17.10 Choosing a Rendering Model for Our Editor

This is where the theory becomes an architectural decision.

Compare the options:

| Approach                            | Suitability                                                         |
| ----------------------------------- | ------------------------------------------------------------------- |
| Direct incremental terminal updates | Simple initially, increasingly difficult to reason about            |
| Retained visual hierarchy           | Excessive complexity for this editor                                |
| Immediate full-frame rendering      | Simple and predictable                                              |
| Application-side screen buffer      | Useful when frame construction needs an intermediate representation |

The important conclusion should be nuanced:

**Our editor uses full redraw as its rendering strategy**, as established earlier by ADR-002. The reason is simplicity and predictability.

But:

> Full redraw does not mean the renderer must write every visual decision directly to the terminal as it computes it.

That distinction is what justifies Chapter 18.

---

# 17.11 From Rendering Theory to a Screen Buffer

This should be the bridge to the next chapter.

Conceptually:

```text
Editor State
     │
     ▼
Frame Construction
     │
     ▼
Screen Buffer
     │
     ▼
Terminal Output
```

Explain that the **Screen Buffer is an intermediate representation of the frame**.

Don't explain its data structure yet.

Don't introduce dirty flags yet.

Don't explain how lines are stored.

Those belong in Chapter 18.

This section should answer only:

> Why might a renderer benefit from constructing a frame before sending it to the terminal?

---

# 17.12 Design Alternatives

Briefly evaluate alternatives specifically for this project:

### Alternative 1 — Write directly to the terminal

```text
State → Terminal
```

Simple, but mixes frame construction with terminal output.

### Alternative 2 — Retained UI model

```text
State → Persistent UI Tree → Terminal
```

Powerful, but unnecessary complexity.

### Alternative 3 — Application-side frame representation

```text
State
  ↓
Frame
  ↓
Terminal
```

Chosen because it provides a clean boundary without introducing a complete retained-mode UI system.

This section should be concise because the major rendering strategy was already recorded in the ADRs.

---

# 17.13 Complexity and Trade-offs

Discuss rendering from an engineering perspective.

Topics:

- full-frame construction cost
- terminal output cost
- memory required for a frame
- simplicity versus optimization
- predictable behavior versus incremental rendering
- why the chosen approach is appropriate for the project's scope

Avoid prematurely analyzing screen-buffer algorithms. Chapter 18 will do that.

The key conclusion:

> For a small terminal editor, reducing architectural complexity is more valuable than minimizing every rendering operation.

---

## Sections I would **not** include

To keep the chapter genuinely non-duplicative, I would explicitly avoid sections on:

- ANSI escape sequences — Chapter 4 already covers them.
- Raw terminal output — Chapters 4–6.
- Event loop implementation — Chapter 9 already defines it.
- Editor State design — Chapter 12.
- Cursor rendering rules — Chapter 14 and later Chapter 19.
- Input-to-command flow — Chapters 15–16.
- Full-redraw justification as a new decision — Chapter 11 / ADR-002.
- Status bar/message bar implementation — Chapter 19/21.
- Viewport and scrolling implementation — Chapters 19–20.
- Dirty-state semantics — already covered by the command/state architecture and will be used later rather than redefined here.
- Actual screen-buffer data structures — Chapter 18.
- Actual renderer implementation — Chapter 19.

### Recommended final structure

**Chapter 17 — Rendering Theory**

1. Introduction
2. Learning Objectives
3. The Rendering Problem
4. What Is a Frame?
5. Immediate-Mode Rendering
6. Retained-Mode Rendering
7. Rendering and the Game Loop
8. Double Buffering
9. Rendering Models in a Terminal
10. Choosing a Rendering Model for Our Editor
11. From Rendering Theory to a Screen Buffer
12. Design Alternatives
13. Complexity and Trade-offs
14. Common Mistakes
15. Summary
16. Preview of the Next Chapter

This is, in my view, a stronger Chapter 17 than simply following the original four-item outline. It preserves all four planned topics while giving the chapter a **clear architectural purpose**: establish the theory needed to understand why Chapter 18 introduces a Screen Buffer, without prematurely teaching Chapter 18 or repeating Chapters 9–16. The architecture already explicitly defines the Renderer as reading `Editor State`, generating a complete visual frame, and not modifying the document, so this chapter can build directly on that contract.
