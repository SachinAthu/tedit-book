# Chapter Summary

The **Cursor** answers a single question: _where in the Document is the user currently positioned?_ It represents a **logical position** in the Document, not the blinking character on the terminal — the terminal cursor is only a visual representation produced by the Renderer much later.

That logical position is described by two coordinates, `cursor_row` and `cursor_column`. The row identifies a line in the Document, and the column identifies a position within that line. Because the column points _between_ characters, a line of five characters has six valid positions, and the end of a line is just as valid as its beginning.

The Cursor must always be valid: `0 <= cursor_row < line_count` and `0 <= cursor_column <= current_line_length`. Every movement operation preserves these invariants rather than leaving another component to repair an invalid position afterward.

Horizontal movement is local to the current line: Left and Right change the column, stop at the line boundaries, and update the preferred column, because the user is deliberately establishing a new horizontal intention. Vertical movement is more subtle because lines have different lengths: Up and Down change the row while attempting to preserve the preferred column, clamping the actual column to the destination line when it is shorter. The preferred column is navigation state — it records the horizontal position the user intends to maintain while moving vertically.

Finally, the Cursor lives in Document coordinates, while the terminal can display only a portion of the Document at a time. The Viewport determines which part is visible and keeps the cursor on screen, and the Renderer translates logical coordinates into screen coordinates. The Cursor remains a logical concept throughout: its behavior belongs entirely to the Cursor module, while the Command Engine decides when cursor operations occur.

## Cursor Movement Rules

The Cursor's movement behavior can now be expressed as a compact contract.

| Operation |         Row |     Current Column | Preferred Column |
| --------- | ----------: | -----------------: | ---------------- |
| Left      |   unchanged |        −1, bounded | update           |
| Right     |   unchanged |        +1, bounded | update           |
| Up        | −1, bounded | preferred, clamped | preserve         |
| Down      | +1, bounded | preferred, clamped | preserve         |

These rules form the behavioral contract that the Cursor implementation will later follow.

## Exercises

### Review Questions

- What is the difference between a logical cursor and a terminal cursor?
- Why are there potentially six cursor positions in a five-character line?
- Why is preferred column necessary?
- Why shouldn't the Renderer own cursor movement?
- Why must cursor coordinates be validated?

### Thought Experiments

Give the reader examples such as:

```text
Hello
Hi
Hello World
```

and ask them to determine the cursor position after a sequence of `Down`, `Down`, `Up`, `Right`, etc.

### Design Exercise

Ask the reader to define the invariants that must always be true for a valid `Cursor`.

This fits the book's goal of developing independent understanding rather than simply reproducing implementation.

## Common Mistakes

- Treating the terminal cursor as the editor cursor.
- Allowing the cursor to move outside the document.
- Losing the preferred column during vertical movement.
- Assuming every line has the same length.
- Mixing document coordinates with screen coordinates.
- Putting cursor movement logic directly into the Renderer.
- Making the Command Engine responsible for low-level cursor calculations.
- Forgetting empty lines and empty documents.
- Implementing scrolling as part of terminal cursor movement.

## Preview of the Next Chapter

The next chapter moves from **where the user is positioned** to **how the editor understands what the user wants to do**. Chapter 15 introduces the **Input Engine**, which receives raw terminal input and transforms it into meaningful key events that the rest of the editor can understand. We will examine input events, key mapping, escape sequences, and the boundary between low-level terminal input and higher-level editor commands. By the end of the chapter, the input path will be clearly separated from the Cursor and Command Engine, completing another important boundary in the editor's architecture.
