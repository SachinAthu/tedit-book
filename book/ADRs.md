# ADR-001

Programming Language

Decision:

C

Reason:

Lowest abstraction.

---

# ADR-002

Rendering Strategy

Decision:

Full redraw

Reason:

Simpler implementation. Redrawing the entire screen on every frame avoids the complexity of tracking incremental changes and keeps the renderer a pure function of editor state.

First referenced: Chapter 4, Section 4.5 (Clearing the Screen)

---

# ADR-003

Terminal Input Mode

Decision:

Raw mode (non-canonical)

Reason:

A text editor must respond to individual key presses — including arrow keys, Backspace, Delete, Tab, Escape, and keyboard shortcuts — without waiting for the Enter key. Canonical mode's line buffering, echo, and signal generation prevent this. Raw mode disables these behaviors and delivers every byte directly to the application, making it the only viable input mode for an interactive text editor.

Decision:

Save and restore original terminal settings

Reason:

Raw mode is a temporary operating state, not a permanent configuration change. The editor must save the original `termios` settings before enabling raw mode and restore them before exiting — whether the exit is normal or due to an error. Failing to restore the settings leaves the user's terminal in an unusable state (no echo, no line editing, broken shell). This save-modify-restore pattern applies broadly: any software that temporarily modifies shared system resources should leave them exactly as it found them.

First referenced: Chapter 5

---

# ADR-004

Input Parsing Architecture

Decision:

Separate raw byte parsing from editor logic by introducing a dedicated Input Parser component.

Reason:

Terminal input arrives as an unstructured byte stream containing printable characters, control characters, multi-byte UTF-8 sequences, and variable-length escape sequences — all sharing common prefixes (e.g., ESC). Mixing byte-level parsing directly into editor commands couples the editor to terminal protocol details, making the code harder to maintain, test, and port. A dedicated Input Parser sits between the raw byte stream and the Input Engine, translating terminal-specific byte sequences into abstract key events (`KEY_ARROW_UP`, `KEY_CTRL_S`). This provides a clean boundary: the parser handles terminal protocols; everything above it operates on structured events. The design follows the broader principle of separating low-level implementation details from high-level application behavior. (The Input Engine that consumes these key events is a distinct component between the Input Parser and the Command Engine, introduced later by ADR-015.)

First referenced: Chapter 8

# ADR-005

Layered Architecture

Decision:

Organize the editor into three logical layers:

- Application
- Core
- Platform

Reason:

Separates responsibilities and reduces coupling.

First referenced: Chapter 8

# ADR-006

Centralized Editor State

Decision:

Maintain a single EditorState structure that acts as the application's single source of truth.

Reason:

Avoid duplicated state and inconsistent updates.

First referenced: Chapter 8

# ADR-007

State Ownership

Decision:

Each piece of state has exactly one owning component.

Reason:

Improves consistency and simplifies debugging.

First referenced: Chapter 8

# ADR-008

Dependency Direction

Decision:

Dependencies flow toward the Core Layer. Circular dependencies are prohibited.

Reason:

Produces a modular architecture with low coupling.

First referenced: Chapter 8

# ADR-009

Component Responsibilities

Decision:

Each architectural component has one primary responsibility.

Reason:

Follows the Single Responsibility Principle and improves maintainability.

First referenced: Chapter 8

# ADR-010

Module-Based Directory Layout

Decision:

Organize the project into module directories — one per architectural component — with each module's header and source files co-located. Reject the alternative of grouping files by type (all headers in one directory, all sources in another), and reject a flat layout (all `.c` and `.h` files in a single directory).

Reason:

A module-based layout makes the architecture visible in the directory tree. Every architectural component from Chapter 8 becomes a physical directory whose name communicates its responsibility: `input/`, `renderer/`, `document/`, and so on. Co-locating each module's header and source file keeps its public interface and implementation together, making it easier to find, understand, and modify. A flat layout scales poorly as modules are added, and a file-type layout separates a module's interface from its implementation, requiring the developer to navigate two unrelated trees to understand one component.

First referenced: Chapter 10

# ADR-011

Dedicated Common Module

Decision:

Create a `common/` directory with a `common.h` header for shared type definitions, constants, enumerations, and utility macros that are needed by multiple independent feature modules. Explicitly prohibit placing module-specific logic (document operations, rendering functions, cursor movement, file handling) into this module.

Reason:

Without a central shared module, definitions needed by multiple components would either be duplicated — leading to inconsistencies when one copy changes — or placed in an arbitrary module's header, creating an unnecessary dependency. The common module eliminates both problems by providing a single source of truth for genuinely shared infrastructure. The prohibition against module-specific logic prevents the common module from becoming a dumping ground that introduces the very coupling it is intended to eliminate.

First referenced: Chapter 10

# ADR-012

CMake as Build System

Decision:

Use CMake as the project's build system rather than traditional Make, Meson, Ninja, Bazel, or hand-written build scripts.

Reason:

CMake provides cross-platform support (Linux, macOS, Windows, embedded platforms), scales from small educational projects to large production systems, integrates with modern IDEs, and generates native build files from a single project description. This allows the same `CMakeLists.txt` to describe the project regardless of the target platform, avoiding platform-specific build scripts. While CMake introduces its own configuration language, the investment in learning it is offset by eliminating the need to manually track dependencies, compiler flags, and incremental rebuilds as the project grows.

Trade-off: CMake adds an external dependency and a configuration language to learn. For a single-file program, a compiler command would suffice. However, the editor will grow to multiple modules, each with their own source and header files, making manual compilation impractical. CMake is introduced early so that the build system scales with the project rather than being retrofitted later.

First referenced: Chapter 10

---

# ADR-013

Document Representation

Decision:

Represent the document in memory as a **vector of lines** — a collection of independently stored text lines, each holding its own sequence of characters — rather than a single string, a gap buffer, a piece table, or a rope.

Reason:

A text editor constantly works with lines: the cursor has a row and column, the renderer draws one line at a time, pressing Enter splits a line, and Backspace at the start of a line merges it with the previous one. A vector of lines maps these concepts directly onto the data structure, making line operations natural and easy to reason about. It is simple to visualize and debug, matches the cursor's row/column model and the renderer's line-by-line output, and is consistent with the project's requirements, which prioritize simplicity, readability, and educational value over advanced optimization. More sophisticated representations such as gap buffers, piece tables, and ropes offer better performance for certain workloads but hide the text behind additional layers of machinery that are unnecessary for this editor's scope.

Trade-off: The vector of lines does not eliminate the fundamental cost of editing contiguous character sequences. Character insertions and deletions still shift memory within the affected line, so very long lines can be costly to edit, and inserting or deleting lines can require changes to the underlying collection. These limitations are acceptable for the documents our editor is designed to handle. Because the Document's responsibility is well defined (owning the text and nothing else), the representation can be replaced later without changing the higher-level architecture.

First referenced: Chapter 13

---

# ADR-014

Cursor Movement Model

Decision:

Model the cursor as a logical position — a row and a column — managed by a dedicated Cursor component whose movement is explicit and local. Horizontal movement (Left/Right) is confined to the current line and stops at its boundaries rather than automatically crossing into adjacent lines; vertical movement (Up/Down) changes the row while attempting to preserve a separate preferred column, clamping the actual column to the destination line's length when that line is too short. The column is a logical character position in Document coordinates — one per Unicode code point, including TAB — never a terminal screen column or a UTF-8 byte offset; visual expansion of characters such as TAB is a Renderer concern.

Reason:

Text documents have lines of different lengths, so the cursor cannot naively preserve a single column across vertical movement. Storing only the current column is insufficient: after passing through a short line, the cursor would permanently drift toward the beginning of the line and never return to the user's intended horizontal position. A preferred column records the user's horizontal intent, while the actual column reflects the constraint imposed by the current line. Likewise, confining horizontal movement to a single line keeps the operation's meaning simple — crossing into another line is a separate movement decision, made explicitly rather than as an accidental side effect of incrementing a column.

Trade-off: The preferred column is additional navigation state that must be kept consistent — horizontal movement establishes it, vertical movement preserves it, and editing operations define their own effect on it. This is more complex than tracking a single column, but it is what makes vertical navigation through lines of different lengths behave predictably.

First referenced: Chapter 14

---

# ADR-015

Input Engine

Decision:

Introduce a dedicated Input Engine component between the Input Parser and the Command Engine. The Input Engine maps abstract key events to editor commands and dispatches those commands for execution, without reading raw terminal bytes, parsing escape sequences, or modifying the Document.

Reason:

The Input Parser understands terminal input, and the Command Engine understands how to execute editor operations. Deciding which command a key event represents is a third, distinct responsibility. Folding it into the Input Parser would couple terminal-specific parsing to editor-specific commands; folding it into the Command Engine would couple command execution to the keyboard layout. A dedicated Input Engine completes the input pipeline as three clean stages — read bytes (Input Handler), parse into events (Input Parser), map to commands (Input Engine) — giving each stage one responsibility. It also keeps commands independent of the keys that trigger them: a shortcut is a binding between an input event and a command, so bindings can change (including future configurable shortcuts) without changing the commands themselves.

Trade-off: The Input Engine adds one component and one boundary to the architecture for a mapping that a small editor could perform with a simple `switch` statement. That structural cost is accepted because the boundary — separating input from behavior — is what keeps input handling from accumulating document, cursor, and file responsibilities as the editor grows.

First referenced: Chapter 15

---

# ADR-016

Command Engine

Decision:

Introduce a dedicated Command Engine component that receives commands from the Input Engine, executes them, and coordinates the resulting state changes. Represent each command as a simple value — a command type (an enumeration) plus optional command data — and dispatch commands using a straightforward `switch` statement over the command type, rather than a function-pointer table or a command-object framework.

Reason:

The Input Engine answers "which command does this input represent?", while the Command Engine answers "what should happen when this command executes?". These are distinct responsibilities. Folding command execution into the Input Engine would couple input interpretation to editor behavior, causing the input layer to accumulate knowledge of the Document, Cursor, and File Manager. Distributing command handling across the core components would leave no single place to coordinate the multi-component state transitions that commands such as `INSERT_CHARACTER` require. Representing a command as a value — a type plus optional data — keeps it independent of the input that triggered it, so a shortcut or a future menu system can reach the same command without changing its implementation, while the Document and Cursor retain ownership of their own rules.

Trade-off: The Command Engine adds one component and one boundary for work that a small editor could perform with a `switch` statement in the input path. The simple value representation and switch dispatch are sufficient for the editor's small, fixed command vocabulary, but they do not provide undo/redo, command history, or dynamic registration — capabilities that would require the full Command Pattern, which is deliberately deferred to Chapter 32.

First referenced: Chapter 16

---

# ADR-017

Rendering Model and Frame Representation

Decision:

Use immediate-mode rendering combined with an application-side frame representation. The renderer derives a complete visual frame from the current Editor State on each frame, renders once per event-loop iteration (event-driven rather than a continuous game loop), and represents the completed frame in an in-memory Editor Screen Buffer before converting it into terminal output.

Reason:

Immediate-mode rendering keeps the renderer a read-only consumer of the Editor State: given the same relevant state, the renderer produces the same visual result, with no second persistent visual model to keep synchronized. Rendering once per event reflects the editor's event-driven architecture and avoids the wasted work of a continuously running game loop. Representing the frame in an Editor Screen Buffer separates frame construction from terminal output, so the terminal never observes a partially constructed frame, without turning the rendering system into a full retained-mode framework.

Trade-off: A retained-mode system could avoid reconstructing unchanged regions of the display, but for a small terminal editor the additional state and synchronization complexity is not justified. The Editor Screen Buffer adds a modest, bounded amount of memory proportional to the visible terminal area, accepted in exchange for a simpler, more predictable rendering pipeline.

First referenced: Chapter 17

---

# ADR-018

Screen Buffer Representation

Decision:

Represent the Screen Buffer as a reusable in-memory grid of **screen cells** — one cell per visible terminal position — stored as a contiguous one-dimensional allocation addressed with `index = row × columns + column`. Reject direct terminal output, a string-based frame buffer, separate character and attribute buffers, and a dirty-region buffer as the baseline representation.

Reason:

A terminal screen is a grid of positions, so a cell per position maps directly onto that structure and matches the renderer's row/column access. Each cell can hold the character plus optional visual attributes (foreground, background, attributes) for one position, which a string-based buffer cannot express cleanly. A contiguous one-dimensional allocation preserves the two-dimensional logical model while keeping storage simple and predictable, with memory bounded by terminal dimensions (`O(rows × columns)`) rather than document size. Because the editor uses full redraw, the buffer only needs to represent the current frame, so the same allocation can be reused across rendering cycles — keeping allocation out of the normal rendering path.

Trade-off: The cell buffer stores a cell for every visible position even when much of the screen is empty or unchanged, and full redraw regenerates the entire frame rather than updating only changed regions. These costs are accepted because terminal screens are small and the project prioritizes a simple, predictable rendering pipeline over incremental-rendering optimizations; dirty-region tracking remains a deferred optimization if profiling later justifies it.

First referenced: Chapter 18

---

# ADR-019

Viewport and Scrolling Model

Decision:

Represent the visible region of the document with a Viewport that stores only two integer offsets — `viewport_row`, the first visible document row, and `viewport_column`, the first visible **visual/display column** of the rendered viewport — rather than a copy of the document. Scrolling keeps the Cursor visible by adjusting these offsets, and the editor uses horizontal scrolling (moving `viewport_column` to expose different visual columns of a single long line) instead of soft wrapping. The decision distinguishes the two coordinate spaces explicitly: vertical scrolling is based on logical document rows; horizontal scrolling is based on visual/display columns; `viewport_column` represents the first visible visual column; cursor visibility is determined using the cursor's rendered visual position, not its logical column; the Cursor's logical column remains a Document coordinate; and the Renderer bridges logical positions and visual positions. `viewport_column` is not a UTF-8 byte offset, not a logical Document character index, and not `Cursor.column` — the values coincide only when the rendered line contains no variable-width characters.

Reason:

The terminal can display only a portion of the document, so the editor must record which portion is visible. Storing just two offsets keeps the Viewport's state constant (`O(1)` space), avoids duplicating the document, and makes keeping the cursor visible a matter of a few coordinate comparisons (`O(1)` time). The scrolling system operates on the rendered coordinate space because its purpose is to determine which portion of the terminal screen is visible:

```text
Document / Cursor ── logical position ──► Renderer ── visual position ──► Viewport / scrolling
```

Rows and columns therefore follow different coordinate spaces. Document rows map one-to-one to screen rows, so `viewport_row` can count logical rows directly. Horizontal positions do not: a logical column counts Unicode code points, while the terminal displays cells, and characters such as TAB occupy several cells (Chapters 14 and 19). The Renderer owns that logical-to-visual mapping — including the configured tab width — and `viewport_column` is expressed in the visual columns it produces. Scrolling never performs UTF-8 decoding, never computes tab widths, and never modifies the Document; it only consumes the rendered coordinates. On lines without variable-width characters, logical and visual columns coincide, which is why the simpler single-space model appears to work until TABs are introduced. Horizontal scrolling preserves the simple row/column coordinate model established by the Cursor and the Renderer, maintaining a one-to-one mapping between document rows and screen rows. Soft wrapping would break that mapping by spreading a single document row across multiple screen rows, requiring the editor to calculate wrapped screen lines, determine the cursor's visual row, and map screen positions back to document positions. The project requirements also explicitly exclude soft wrapping.

Trade-off: Horizontal scrolling means a very long line cannot be viewed in full at once — the user scrolls horizontally to see later portions of the line, whereas soft wrapping would show the entire line across multiple rows. The two-offset model likewise provides no smooth scrolling, scroll margins, split windows, or independently scrolling views; those features become justified only when the interface itself grows more complex. These costs are accepted for the baseline editor, which prioritizes simplicity, predictable behavior, and educational value over advanced interface behavior.

First referenced: Chapter 20

---

# ADR-020

File Persistence Model

Decision:

Introduce a dedicated File Manager as the single boundary between the editor and the file system, and adopt a full-document persistence model: loading reads the entire file, decodes its UTF-8 bytes into Unicode code points to build the in-memory Document, and saving encodes those code points back into UTF-8 and rewrites the complete document to disk. Reject incremental persistence, memory mapping, and lazy or partial loading as the baseline strategy.

Reason:

The File Manager gives file persistence a single, narrow responsibility (ADR-009), so POSIX file APIs remain localized to one component rather than spreading through the Command Engine, Document, and Renderer. Loading the whole document keeps the editor's core model simple: the Document in memory is the authoritative text being edited, and the file on disk is merely its persistent image. A full save is easy to understand and reason about — the file always receives the complete current state of the Document, avoiding the bookkeeping needed to track which regions changed. This matches the project's priority on simplicity and educational value.

Trade-off: Full-document persistence ties memory usage to the number of code points and load/save time to the number of bytes (UTF-8 decoding on load, encoding on save), so very large files can be slow to load or save and expensive to hold in memory. Incremental persistence or memory mapping could reduce some of these costs but would introduce substantial additional complexity: tracking modified regions, maintaining consistency between memory and disk, and handling partial updates and interrupted writes. For a small, single-document educational editor those costs are not justified, and the model can be revisited if requirements change.

First referenced: Chapter 21

---

# ADR-021

Text Encoding Strategy

Decision:

Store the Document's text as decoded Unicode code points, one code point per element, arranged line by line. Use UTF-8 only at the file boundary: the File Manager decodes UTF-8 bytes into code points when loading and encodes code points back into UTF-8 bytes when saving. Store logical lines without line-ending bytes, recognizing both LF and CRLF on load and writing a single canonical line ending (LF) on save. Every logical line boundary — including the one after the final line — is serialized as LF, so a saved file always ends with a trailing newline.

Reason:

Storing code points gives the Document a fixed-width element per character, so a cursor column, a line length, and a character position are all measured in code points rather than bytes. This removes the byte/character mismatch from inside the Document — cursor movement, insertion, deletion, and length computation all operate directly on characters. UTF-8 remains the encoding of the file on disk because it is compact, ASCII-compatible, and standard on Linux. Keeping the encoding at the boundary confines byte-level concerns to loading and saving, where the File Manager already owns the transfer.

Trade-off: A code point requires up to four bytes of storage even for ASCII text, which stores as one byte in UTF-8, so the in-memory Document can be larger than the file it came from. The editor also gains a decode step on load and an encode step on save. In exchange, the core editing model becomes simpler and more predictable. Full grapheme-cluster handling (combining marks, emoji sequences) is deliberately out of scope: the editor treats one code point as one logical character.

First referenced: Chapter 22
