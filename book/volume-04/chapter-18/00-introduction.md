# Volume IV — Rendering Engine

# Chapter 18 — Screen Buffer

## Introduction

In the previous chapter, we examined how the editor turns its internal state into a visual frame. We compared different rendering models and established the rendering strategy for our editor: immediate-mode rendering with a full redraw, performed once after each event-loop iteration. The renderer reads the current `EditorState` and produces a complete representation of what should appear on the terminal.

That leaves an important question: **where does this representation exist before it is sent to the terminal?**

The answer is the **Screen Buffer**.

The Screen Buffer is an in-memory representation of the terminal's visible screen. Instead of constructing terminal output directly while examining the document, the editor first builds a complete frame in memory. That frame can then be converted into terminal output as a separate step.

This creates a clear boundary:

```text
Editor State
     │
     ▼
Frame Generation
     │
     ▼
Screen Buffer
     │
     ▼
Terminal Output
     │
     ▼
Terminal
```

The Screen Buffer is not another representation of the document. The **Text Buffer** stores the logical contents of the document, while the **Screen Buffer** represents how those contents should currently appear on the terminal. A single document line may occupy only part of the screen, may be affected by the current viewport, and may be accompanied by interface elements such as the status bar and message bar.

This distinction becomes increasingly important as the editor grows. The document describes _what exists_; the Screen Buffer describes _what is visible_.

The Screen Buffer also gives the renderer a useful intermediate representation. The renderer can construct a complete frame without simultaneously worrying about terminal escape sequences or partially updating the terminal. Once the frame is complete, it can be converted into terminal output.

This chapter examines the structure and lifecycle of that buffer. We will look at what a screen cell represents, how the buffer relates to terminal dimensions and `EditorState`, how a frame is generated, and how the concept of dirty regions differs from the document's modified state.

We will also examine why our editor does **not** need incremental dirty-region rendering as part of its baseline design. The full-redraw strategy established in Chapter 17 deliberately favors a simple, predictable rendering pipeline over the additional state and synchronization required to track individual screen changes.

By the end of the chapter, the Screen Buffer will no longer be an abstract rendering concept. It will be a concrete part of the editor's rendering pipeline and the foundation on which the Renderer introduced in the next chapter will operate.

## Learning Objectives

By the end of the chapter, readers should understand:

- what the Editor Screen Buffer represents
- why it is different from the Document
- why it is different from terminal output
- what information a screen cell needs
- how a complete frame is represented in memory
- how frame generation populates the buffer
- how the buffer relates to terminal dimensions
- what dirty tracking means
- why dirty tracking is not part of the baseline rendering strategy
- how the Screen Buffer prepares the design for the Renderer
