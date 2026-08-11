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

Terminal input arrives as an unstructured byte stream containing printable characters, control characters, multi-byte UTF-8 sequences, and variable-length escape sequences — all sharing common prefixes (e.g., ESC). Mixing byte-level parsing directly into editor commands couples the editor to terminal protocol details, making the code harder to maintain, test, and port. A dedicated Input Parser sits between the raw byte stream and the editor's Command Engine, translating terminal-specific byte sequences into abstract key events (`KEY_ARROW_UP`, `KEY_CTRL_S`). This provides a clean boundary: the parser handles terminal protocols; everything above it operates on structured events. The design follows the broader principle of separating low-level implementation details from high-level application behavior.

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
