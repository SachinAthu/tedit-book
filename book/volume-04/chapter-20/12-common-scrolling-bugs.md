# 20.12 Common Scrolling Bugs

Scrolling logic is simple in principle, but small boundary errors can produce noticeable problems. The most common issues are related to keeping the Cursor and Viewport coordinates consistent.

## Cursor Disappears at the Bottom

A common off-by-one error is treating this position as visible:

```text
cursor_row == viewport_row + visible_rows
```

The last visible row is actually:

```text
viewport_row + visible_rows - 1
```

Therefore, the cursor has already moved outside the viewport when:

```text
cursor_row >= viewport_row + visible_rows
```

## Negative Offsets

Moving upward can accidentally reduce an offset below zero:

```text
viewport_row = -1
```

Both viewport offsets must remain non-negative:

```text
viewport_row >= 0
viewport_column >= 0
```

## Horizontal Scrolling Never Reverses

An editor may correctly increase `viewport_column` when the cursor moves right but fail to decrease it when the cursor moves back toward the beginning of the line.

The result is that the cursor can move left logically while the visible text remains horizontally displaced.

## Horizontal Scrolling Ignores Tab Expansion

A subtler bug is comparing the cursor's logical column directly against `viewport_column` on lines that contain TAB characters.

Because `viewport_column` counts visual columns while `cursor.column` counts logical code points (Chapters 14 and 19), the two are not comparable before the Renderer's expansion. Consider the line `A<TAB>123456789` from Section 20.4: after the TAB, the cursor is at logical column `2` but visual column `4`. A check written as:

```text
cursor_column >= viewport_column + visible_columns
```

therefore decides at the wrong position and either scrolls unnecessarily or leaves the cursor off screen. The check must use the cursor's visual column obtained from the Renderer's logical-to-visual mapping.

## Scrolling Based on Screen Position

Scrolling decisions should be based on the Cursor's **document coordinates**, not on the position where the Renderer previously displayed it.

The Cursor represents the logical position; the Viewport determines how that position maps to the screen.

## Renderer Owns Scrolling

Another architectural mistake is allowing the Renderer to modify the Viewport when it discovers that the Cursor is outside the screen.

The Renderer should remain a read-only consumer of Editor State. Viewport adjustment belongs to the state-management side of the editor.

## Editing Leaves the Cursor Outside the Viewport

Scrolling should not be associated only with arrow-key commands. Operations such as insertion, deletion, line splitting, and line merging can also change the Cursor's position.

After such operations, cursor visibility should be checked before the next frame is rendered.

The general rule is:

> **Update the Cursor, ensure it is visible, then render.**

This keeps scrolling predictable and prevents different commands from implementing their own inconsistent viewport logic.

## Summary

Most scrolling bugs stem from off-by-one boundary errors, negative offsets, coordinate-space mismatches, or misplaced scrolling responsibility. The rule "update the Cursor, ensure it is visible, then render" keeps the behavior consistent.

In the next section, the **chapter summary** consolidates the Cursor–Viewport model and previews the next chapter.
