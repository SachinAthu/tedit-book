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
