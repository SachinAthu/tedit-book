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
