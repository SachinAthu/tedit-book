# 20.7 Scrolling After Editing

Scrolling is not triggered only by explicit cursor movement commands. Editing operations can also change the cursor's position or the structure of the Document.

For example, pressing **Enter** splits a line:

```text
Before:

Line 1
Line 2| remaining text
Line 3
```

After the operation:

```text
Line 1
Line 2
| remaining text
Line 3
```

The cursor has moved to a new document position. If that position falls outside the visible region, the Viewport must be adjusted.

The same can happen when:

- inserting characters
- deleting characters
- splitting a line
- merging adjacent lines

The important principle is that scrolling should respond to the **resulting editor state**, rather than being tied to particular keyboard commands.

A useful sequence is:

```text
Editing Operation
       │
       ▼
Update Document
       │
       ▼
Update Cursor
       │
       ▼
Ensure Cursor Is Visible
       │
       ▼
Render
```

This keeps the responsibility for scrolling in one place. Individual editing commands do not need to contain separate logic for handling the viewport.

> **Any operation that changes the logical cursor position should be followed by a visibility check.**

This also prevents subtle inconsistencies where cursor movement commands scroll correctly but editing operations leave the cursor outside the visible region.

## Summary

Scrolling should respond to the resulting editor state, not to specific keyboard commands. Any operation that changes the logical cursor position should be followed by a visibility check.

In the next section, we will examine the **boundaries** that constrain the Viewport and keep its offsets within valid document and terminal ranges.
