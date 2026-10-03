# Volume IX — Advanced Topics

# Chapter 42 — Terminal Graphics

## Introduction

## Learning Objectives

## 42.1 From Character Cells to Rich Terminal Displays

Introduce the conceptual shift.

Earlier chapters treated the terminal primarily as:

```text
rows × columns
    ↓
screen cells
    ↓
characters + attributes
```

Advanced terminals can additionally support:

```text
text
colors
images
mouse events
graphics protocols
```

Discuss why this changes the terminal programming model.

**Important:** Do not reteach ANSI escape sequences here. Chapter 4 already established those fundamentals.

---

## 42.2 The Terminal Graphics Model

Define the distinction between:

- character-based rendering
- cell-based styling
- graphical payloads
- terminal protocols
- terminal capabilities

Explain that the editor does not directly control a GPU or framebuffer.

Instead:

```text
Editor
   │
   │ terminal protocol
   ▼
PTY
   │
   ▼
Terminal Emulator
   │
   ▼
Display
```

This reinforces the runtime model rather than duplicating the original terminal introduction.

---

## 42.3 True Color

Introduce 24-bit color and explain how it differs from the color capabilities used by traditional terminals.

Cover:

- indexed colors
- 256-color terminals
- RGB / 24-bit color
- foreground and background color
- color capability negotiation
- terminal limitations

Then connect it to the existing renderer:

```text
Highlight Class
       ↓
Theme
       ↓
Visual Style
       ↓
Screen Cell
       ↓
Terminal Output
```

This should connect naturally to Chapter 32's syntax-highlighting architecture without repeating the highlighting discussion.

---

## 42.4 Terminal Capability Detection

This section is important and should come **before images and mouse support**.

Discuss why an editor cannot simply assume:

> "The terminal supports feature X."

Cover the general problem of:

- capability detection
- environment information
- terminal identification
- feature negotiation
- graceful fallback
- unsupported features

The key architectural principle:

> **Optional terminal capabilities must never become hidden assumptions inside the Core.**

This connects directly to the platform-boundary decisions in the architecture. The Terminal Interface already owns terminal-specific behavior. 03-system-architecture

---

## 42.5 Images in Terminals

Introduce terminal image display conceptually.

Explain:

- why an image cannot be represented by ordinary screen cells
- transmitting image data through the terminal protocol
- image dimensions
- placement
- terminal-side image storage
- redraw and refresh concerns
- image lifetime

A useful conceptual distinction:

```text
Text rendering

Document
   ↓
Screen Buffer
   ↓
Cells
   ↓
Terminal


Image rendering

Image Data
   ↓
Terminal Graphics Protocol
   ↓
Terminal-side image
   ↓
Display
```

Do **not** turn this into an implementation tutorial.

The purpose is to explain why image rendering represents a fundamentally different rendering path.

---

## 42.6 The Kitty Graphics Protocol

Now introduce Kitty as a concrete example of a modern terminal graphics protocol.

Cover:

- why a specialized protocol is necessary
- image transmission
- image placement
- image identifiers
- protocol commands
- terminal-side resources
- protocol limitations
- capability dependence

The important lesson should be:

> Kitty is an example of a terminal graphics protocol, not a universal terminal API.

This prevents the chapter from accidentally presenting Kitty-specific behavior as standard terminal behavior.

---

## 42.7 Mouse Input

This should be treated primarily as an **input protocol**, not graphics.

Explain:

- mouse buttons
- press/release events
- movement
- coordinates
- modifier keys
- scrolling
- terminal-to-application event encoding

Then connect it to the existing input pipeline:

```text
Mouse
  ↓
Terminal Emulator
  ↓
PTY
  ↓
Input Handler
  ↓
Input Parser
  ↓
Mouse Event
  ↓
Input Engine
  ↓
Command
```

This is important because Chapters 5–6 already established terminal input and parsing. Here we study **what changes when mouse events are introduced**, rather than re-explaining raw input.

---

## 42.8 Mouse Coordinates and Editor Coordinates

This deserves its own section because it introduces an interesting architectural problem.

A mouse event provides something like:

```text
terminal row
terminal column
```

But the editor needs to determine:

```text
Window
    ↓
Viewport
    ↓
Document row
    ↓
Document column
```

For example:

```text
Terminal Coordinate
        ↓
Window Geometry
        ↓
Viewport Offset
        ↓
Document Coordinate
```

This section can reinforce the coordinate-space distinctions established in the Cursor and Viewport chapters without repeating their definitions.

---

## 42.9 Graphics and the Rendering Architecture

Now answer the architectural question:

> **How would graphical capabilities fit into our editor?**

Compare the baseline renderer:

```text
Editor State
     ↓
Screen Buffer
     ↓
Terminal Interface
```

with an extended renderer:

```text
                 ┌─ Screen Cells ──────► Terminal
Editor State ────┤
                 └─ Graphics Objects ──► Graphics Protocol
```

Discuss whether images belong in:

- the Document
- the Screen Buffer
- the Renderer
- the Terminal Interface
- a separate graphics subsystem

The architectural conclusion should be that **terminal graphics are presentation/platform concerns, not Document concerns**.

This is particularly valuable because it reinforces the ownership boundaries already established in the book.

---

## 42.10 Why Graphics Complicate Full Redraw

This section is worth including because it directly challenges an earlier architectural decision.

The baseline renderer uses:

> full redraw + reusable Screen Buffer

ADR-017 establishes immediate-mode rendering and ADR-018 establishes the reusable cell-based Screen Buffer. 07-adrs

But graphics introduce complications:

```text
Text:
screen cell → overwritten easily

Image:
terminal-side resource
       ↓
placement
       ↓
potentially persistent state
```

Discuss:

- image persistence
- stale graphical content
- clearing behavior
- redraw ordering
- resizing
- scrolling
- terminal state
- synchronization between cell rendering and graphics

This is an excellent advanced architectural discussion because it shows **where the simplicity of the baseline renderer begins to encounter limits**.

---

## 42.11 Terminal Graphics and Portability

This should not repeat Chapter 37's general portability discussion.

Instead, focus narrowly on **capability fragmentation**:

```text
Terminal A → feature supported
Terminal B → feature unsupported
Terminal C → different protocol
Terminal D → partial support
```

Discuss:

- standard vs vendor-specific protocols
- feature detection
- fallback rendering
- optional capabilities
- portability cost

The important distinction:

> Chapter 37 explains how to contain platform differences.  
> Chapter 42 examines why terminal capabilities themselves are fragmented.

That keeps the two chapters complementary rather than repetitive.

---

## 42.12 Designing Graceful Fallbacks

This is a practical architectural consequence of the previous section.

For example:

```text
Image requested
      │
      ▼
Graphics capability?
   /       \
 yes       no
  │         │
image      text
render     fallback
```

Discuss:

- feature degradation
- text-only fallback
- disabling optional features
- preserving core editor functionality
- avoiding fatal failures for optional capabilities

This principle is broader than graphics and is worth teaching as an engineering pattern.

---

## 42.13 Security and Resource Considerations

A short but valuable advanced section.

Discuss:

- untrusted image data
- terminal escape sequences
- resource exhaustion
- large graphical payloads
- terminal implementation vulnerabilities
- clipboard/image-like protocols
- why terminal output should not blindly trust document content

This should remain conceptual rather than becoming a security chapter.

---

## 42.14 When Should Our Editor Support Terminal Graphics?

Bring the discussion back to **our editor**.

Evaluate:

| Capability            | Baseline Editor             |
| --------------------- | --------------------------- |
| 24-bit color          | Optional future enhancement |
| Images                | Deferred                    |
| Mouse                 | Deferred                    |
| Kitty graphics        | Deferred                    |
| Capability detection  | Future platform feature     |
| Text rendering        | Required                    |
| ANSI terminal control | Required                    |

The conclusion should be that **none of these features belongs in the baseline implementation volume unless requirements change**.

This is consistent with the SRS, which explicitly excludes mouse support and other advanced features from the initial editor. 02-software-requirements

---

## 42.15 Architectural Lessons

End with the broader lessons:

1. A terminal is a protocol endpoint, not a framebuffer.
2. Terminal capabilities are not universally available.
3. Graphics introduce resources beyond ordinary screen cells.
4. Input and rendering protocols should remain separated from editor logic.
5. Optional capabilities require explicit detection and fallback.
6. Vendor-specific protocols should remain behind platform/protocol boundaries.
7. The Core should remain independent of terminal graphics.
8. A simple architecture should not be complicated merely to support optional capabilities.

---
