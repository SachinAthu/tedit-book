# Volume IV — Rendering Engine

# Chapter 19 — Renderer

## Introduction

In the previous chapter, we established the **Screen Buffer**: an in-memory representation of the terminal's visible screen. We examined what a screen cell represents, how the buffer relates to terminal dimensions and `EditorState`, and how a complete frame is generated and converted into terminal output.

That leaves an important question: **which component decides what goes into each screen cell?**

The answer is the **Renderer**.

The Renderer is the presentation component of the editor. It reads the current `EditorState` — the Document, Cursor, Viewport, current file, dirty flag, status message, and screen size — and constructs the complete visual frame that the Screen Buffer represents. It does not read keyboard input, execute commands, or modify the Document. It is a read-only projection from application state into screen state.

This chapter follows that transformation from beginning to end. We will define the Renderer's responsibility boundary, then trace how it turns `EditorState` into a visual frame: drawing the document area, translating document coordinates into screen coordinates, rendering the Status Bar and Message Bar, and positioning the Cursor. We will also examine the order in which the Renderer constructs a frame, the design alternatives for organizing its implementation, and the complexity and trade-offs of full-frame rendering.

Throughout the chapter, we treat the Renderer as a **pure function of the editor's state**. Given the same `EditorState`, the Renderer produces the same visual frame. This property keeps the rendering pipeline predictable and forms the foundation on which scrolling, covered in the next chapter, will operate.

## Learning Objectives

By the end of the chapter, readers should understand:

- what the Renderer is responsible for
- why rendering must be read-only
- how the Renderer differs from the Screen Buffer
- how `EditorState` becomes a complete visual frame
- how the document area is rendered
- how document coordinates are translated into screen coordinates
- how the Status Bar is rendered from editor state
- how the Message Bar is rendered and cleared
- how the Cursor is translated into a terminal position
- the order in which the Renderer constructs a frame
- how the Renderer implementation is organized
- the time, space, and trade-off costs of full-frame rendering
