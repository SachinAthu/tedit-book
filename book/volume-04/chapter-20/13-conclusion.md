# Chapter Summary

Scrolling answers a single question left open by the Renderer: **how does the visible region move through the document as the cursor moves?**

The chapter began by separating the coordinate systems involved. **Document coordinates** describe a logical position within the text — a row and column that do not change when the document scrolls. **Screen coordinates** describe where that position appears within the terminal. **Visual/display columns** sit between the two horizontally: the Renderer maps logical columns to visual columns, expanding TAB characters across several cells (Chapter 19). The **Viewport** connects these spaces through its offsets: `document_row = viewport_row + screen_row` and `visual_column = viewport_column + screen_column`.

The Viewport does not copy the document. It stores just two values — `viewport_row` and `viewport_column` — that identify where the visible region begins. Vertical scrolling changes `viewport_row` to control which document rows are visible; horizontal scrolling changes `viewport_column` to control which visual columns of a long line are visible. The editor uses horizontal scrolling rather than soft wrapping, so a long line remains a single document line and the viewport simply exposes different portions of it.

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
   │      │
   │      │ Renderer: logical → visual
   │      ▼
   │  visual position
   │
   └── Viewport
          │
          ├── viewport_row     (document rows)
          └── viewport_column  (visual columns)
                 │
                 ▼
          visible region
                 │
                 ▼
             Renderer
```

The most important takeaway:

> **The Cursor identifies a position in the Document. The Viewport determines which part of that Document is visible. Scrolling changes the Viewport so that the Cursor remains visible — vertically in document rows, horizontally in visual columns, with the Renderer bridging the two.**

The central design choice — representing the Viewport as two offsets (`viewport_row` and `viewport_column`) and using horizontal scrolling instead of soft wrapping — is recorded as **ADR-019: Viewport and Scrolling Model**. The Viewport's offsets are part of the Editor State whose single-writer ownership was established in **ADR-007: State Ownership**, and the scrolling model renders each frame within the full-redraw strategy of **ADR-002: Rendering Strategy**.

## Exercises

### Review Questions

1. What is the difference between document coordinates and screen coordinates?
2. Why does the editor need both `viewport_row` and `viewport_column`?
3. How do `viewport_row` and `viewport_column` together identify the visible region of the document?
4. Why is scrolling not the same as cursor movement?
5. What boundaries constrain the Viewport's offsets, and why must they stay non-negative?
6. Why should the Renderer not modify the Viewport?
7. Why must horizontal scrolling use the cursor's visual column rather than its logical column?
8. What is the time and space complexity of keeping the cursor visible?

### Thought Experiments

1. If the editor adopted soft wrapping instead of horizontal scrolling, how would the mapping between document coordinates and screen coordinates change?

### Design Exercise

1. Write pseudocode for keeping the cursor vertically visible, using `cursor_row`, `viewport_row`, and `visible_rows`.

### Optional Challenge

1. Design a viewport algorithm that keeps one blank row visible below the cursor when possible.

## Preview of the Next Chapter

Chapter 20 completes the Rendering Engine by explaining how the **Viewport** keeps the logical Cursor within the visible terminal region. The next volume moves from displaying the document to persisting it.

Chapter 21 will examine how the editor moves document data between memory and the file system:

- Loading a document from disk
- Saving the in-memory Document
- Handling file I/O errors
- Permissions and failure cases
- Considerations when working with large files

The central transition is:

```text
Chapter 20
Document
   │
   ▼
Viewport
   │
   ▼
Renderer
   │
   ▼
Terminal

Chapter 21
Document
   │
   ▼
File Manager
   │
   ▼
File System
```

The key architectural distinction will remain:

> **The Document is the editable in-memory representation; the file on disk is persistent storage.**

Chapter 21 will therefore introduce file operations without revisiting the Document model or Renderer. The focus shifts from **what the user sees** to **how the editor loads and persists the data being edited**.
