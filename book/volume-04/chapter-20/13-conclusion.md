# Chapter Summary

Scrolling answers a single question left open by the Renderer: **how does the visible region move through the document as the cursor moves?**

The chapter began by separating two coordinate systems. **Document coordinates** describe a logical position within the text — a row and column that do not change when the document scrolls. **Screen coordinates** describe where that position appears within the terminal. The **Viewport** connects the two through its offsets: `document_row = viewport_row + screen_row` and `document_column = viewport_column + screen_column`.

The Viewport does not copy the document. It stores just two values — `viewport_row` and `viewport_column` — that identify where the visible region begins. Vertical scrolling changes `viewport_row` to control which document rows are visible; horizontal scrolling changes `viewport_column` to control which columns of a long line are visible. The editor uses horizontal scrolling rather than soft wrapping, so a long line remains a single document line and the viewport simply exposes different portions of it.

The central responsibility of scrolling is captured by one invariant:

> **After viewport adjustment, the Cursor must be within the visible editing region.**

The Cursor identifies a logical position in the Document; the Viewport determines whether that position is currently visible. Scrolling is therefore not cursor movement. The Viewport changes only when the cursor's position falls outside the visible region, and it changes in response to the resulting editor state rather than to a particular key or command. Cursor movement, text insertion, deletion, line splitting, and line merging all trigger the same visibility check.

The offsets must also remain within valid boundaries. They cannot be negative, and they must not expose a region beyond the available document content. Four boundaries interact — document, cursor, viewport, and terminal — and keeping them separate prevents off-by-one errors when the document or terminal is very small.

Architecturally, the Viewport determines _what_ is visible and the Renderer determines _how_ it is displayed. Scrolling changes the state that defines the visible region; rendering reads that state. The Renderer remains a read-only consumer and never modifies the Viewport.

The chosen model is deliberately simple. Two offsets provide constant-time viewport adjustment, so keeping the cursor visible costs `O(1)` time and the Viewport stores `O(1)` space. Alternatives such as no viewport, a vertical offset only, or soft wrapping were considered and rejected because they either fail to support arbitrary documents or complicate the document-to-screen mapping.

The core model is:

```text
Document
   │
   ├── Cursor
   │      │
   │      ▼
   │  logical position
   │
   └── Viewport
          │
          ├── viewport_row
          └── viewport_column
                 │
                 ▼
          visible region
                 │
                 ▼
             Renderer
```

The most important takeaway:

> **The Cursor identifies a position in the Document. The Viewport determines which part of that Document is visible. Scrolling changes the Viewport so that the Cursor remains visible.**

The Viewport's offsets are part of the Editor State whose single-writer ownership was established in **ADR-007: State Ownership**, and the scrolling model renders each frame within the full-redraw strategy of **ADR-002: Rendering Strategy**.

## Exercises

### Review Questions

1. What is the difference between document coordinates and screen coordinates?
2. Why does the editor need both `viewport_row` and `viewport_column`?
3. How do `viewport_row` and `viewport_column` together identify the visible region of the document?
4. Why is scrolling not the same as cursor movement?
5. What boundaries constrain the Viewport's offsets, and why must they stay non-negative?
6. Why should the Renderer not modify the Viewport?
7. What is the time and space complexity of keeping the cursor visible?

### Thought Experiments

1. If the editor adopted soft wrapping instead of horizontal scrolling, how would the mapping between document coordinates and screen coordinates change?

### Design Exercise

1. Write pseudocode for keeping the cursor vertically visible, using `cursor_row`, `viewport_row`, and `visible_rows`.

### Optional Challenge

1. Design a viewport algorithm that keeps one blank row visible below the cursor when possible.

## Preview of the Next Chapter

This chapter established how the Viewport determines which part of the document is visible. The next question is what the editor communicates _around_ that document:

> We can now scroll through a document, but the editor still does not tell the user which file is open, where the cursor is, or whether changes are unsaved. The next chapter defines the Status Bar and Message Bar.

Chapter 21 moves from **what portion of the document is visible** to **what information the editor communicates around that document**.

It will cover:

- filename
- cursor position
- modified state
- status information
- message display

This also creates a clean progression:

```text
Chapter 19
How the Renderer draws the editor
        ↓
Chapter 20
How the Viewport determines what is visible
        ↓
Chapter 21
How the Status Bar communicates editor state
```
