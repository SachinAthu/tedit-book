# 42.7 Mouse Input

Mouse support introduces a different kind of terminal capability. Unlike true color and terminal graphics, which primarily affect what the terminal displays, mouse support allows the terminal to communicate additional input events back to the application.

This does not change the fundamental input architecture established earlier in the book. Instead, it adds another source of events to the existing pipeline.

The conceptual flow becomes:

```text
Mouse
  │
  ▼
Terminal Emulator
  │
  ▼
PTY
  │
  ▼
Input Handler
  │
  ▼
Input Parser
  │
  ▼
Mouse Event
  │
  ▼
Input Engine
  │
  ▼
Command Engine
```

The important architectural boundary remains the same: terminal-specific representations are handled before higher-level editor logic receives the event.

## Keyboard and Mouse as Input Sources

The baseline editor is keyboard-driven.

A keyboard input eventually becomes an abstract event:

```text
Raw terminal bytes
       ↓
Input Parser
       ↓
KEY_ARROW_UP
       ↓
Input Engine
       ↓
MOVE_CURSOR_UP
```

Mouse support introduces another possible input source:

```text
Raw terminal bytes
       ↓
Input Parser
       ↓
MOUSE_BUTTON_PRESS
       ↓
Input Engine
       ↓
Editor command
```

The Input Engine does not need to know how the terminal encoded the mouse event.

It should receive an abstract event describing what happened.

This preserves the separation established by the input architecture: the Input Handler reads raw bytes, the Input Parser translates terminal-specific sequences into structured events, and the Input Engine maps those events to commands.

## What a Mouse Event Represents

A mouse event can contain several pieces of information.

Conceptually:

```text
Mouse Event
├── event type
├── button
├── terminal row
├── terminal column
└── modifiers
```

The event type may distinguish operations such as:

```text
Button Press
Button Release
Movement
Wheel / Scroll
```

The exact set of events depends on the terminal protocol and the editor's requirements.

The important point is that the Input Parser converts the terminal representation into a structured event before the event reaches the editor's higher layers.

## Mouse Input Is Still Terminal Input

It is tempting to treat mouse support as a separate graphical subsystem.

Architecturally, that would be misleading.

The mouse is physically outside the terminal, but the editor does not receive the mouse directly.

The path is:

```text
Physical Mouse
      │
      ▼
Operating System
      │
      ▼
Terminal Emulator
      │
      ▼
Terminal Protocol
      │
      ▼
PTY
      │
      ▼
Editor
```

The terminal emulator converts the physical interaction into a terminal-level representation.

The editor therefore remains a terminal application.

This is why mouse support belongs naturally within the existing Input Handler and Input Parser architecture.

## Enabling Mouse Reporting

A terminal generally does not have to send mouse events to every application all the time.

The application can request a particular mouse-reporting mode through terminal control mechanisms.

Conceptually:

```text
Editor
  │
  │ request mouse reporting
  ▼
Terminal Emulator
  │
  │ mouse events
  ▼
Editor
```

This introduces an important difference from ordinary keyboard input.

Keyboard input is fundamental to the terminal editor's operation.

Mouse input is an **optional negotiated capability**.

The editor may enable it when mouse interaction is required and disable it when the application exits or no longer needs it.

This follows the same save-modify-restore principle already established for terminal state: temporary terminal configuration should be restored when the editor terminates. ADR-003 applies this principle to raw terminal mode and terminal restoration.

## Mouse Coordinates

A mouse event normally provides coordinates relative to the terminal display.

For example:

```text
Terminal

row = 12
column = 35
```

These coordinates are not automatically document coordinates.

The editor may instead have several coordinate spaces:

```text
Terminal coordinates
        │
        ▼
Window coordinates
        │
        ▼
Viewport coordinates
        │
        ▼
Document coordinates
```

The conversion between these spaces is therefore essential if mouse interaction is used for editing.

This distinction is particularly important because the terminal screen can contain more than the document area.

For example:

```text
+----------------------------------+
| Document Area                    |
|                                  |
|                                  |
|                                  |
+----------------------------------+
| Status Bar                       |
+----------------------------------+
| Message Bar                      |
+----------------------------------+
```

A mouse click on the status bar should not be interpreted as a click on document text.

The event must first be interpreted in the context of the editor's layout.

## Mouse Events and Windows

The multi-window architecture introduced earlier makes coordinate interpretation more interesting.

A terminal coordinate identifies a location on the whole terminal:

```text
Terminal
+-----------------------------+
| Window A | Window B         |
|          |                  |
|          |                  |
|          |                  |
+-----------------------------+
```

The editor must determine which window contains the coordinate.

Conceptually:

```text
Terminal Coordinate
        │
        ▼
Window Hit Test
        │
        ▼
Selected Window
        │
        ▼
Viewport Mapping
        │
        ▼
Document Coordinate
```

This is another reason mouse support should not be implemented as a direct modification to the Document.

The mouse event first passes through the application's existing spatial and state model.

## Mouse Clicks and Editor Commands

Once the editor has determined what the mouse event means, it can translate it into an editor-level operation.

For example:

```text
Mouse Click
    ↓
Terminal Coordinate
    ↓
Window
    ↓
Document Position
    ↓
Move Cursor
```

The Command Engine can then execute the resulting operation.

This maintains the same distinction used for keyboard input:

```text
Input
  ↓
Event
  ↓
Command
  ↓
State Change
```

The command does not need to know whether it was triggered by a keyboard shortcut or a mouse action.

## Mouse Scrolling

Mouse wheel events provide another possible input path to existing editor behavior.

For example:

```text
Mouse Wheel
     ↓
MOUSE_SCROLL_UP
     ↓
Input Engine
     ↓
SCROLL_VIEWPORT_UP
```

The scrolling command remains an editor operation.

The mouse is merely another way of triggering it.

This is an important example of why the separation between input events and commands is valuable.

The same command could potentially be triggered by:

```text
Keyboard
Mouse
Future input method
```

without changing the command's implementation.

This follows directly from the input architecture established by ADR-015, where input events are mapped to commands separately from command execution.

## Mouse Input Does Not Require a New Editor Core

Mouse support therefore does not require a new document representation or a new command architecture.

The existing model remains sufficient:

```text
Mouse
  ↓
Input Parser
  ↓
Input Event
  ↓
Input Engine
  ↓
Command
  ↓
Editor State
```

The editor's Core continues to own:

```text
Document
Cursor
Viewport
Windows
```

The mouse provides another mechanism for manipulating those existing concepts.

This is preferable to introducing mouse-specific state throughout the Core.

## Mouse Support and Capability Detection

As with terminal graphics, mouse support depends on the terminal environment.

The editor therefore needs a capability decision:

```text
Mouse interaction desired
        │
        ▼
Mouse reporting available?
       / \
     yes  no
      │    │
   enable  keyboard
   mouse   operation
```

If mouse reporting is unavailable, the editor should remain usable through its existing keyboard interface.

This is an example of **graceful degradation**.

The optional input mechanism should enhance the editor without becoming a prerequisite for editing.

## Mouse Input and the Baseline Editor

The SRS deliberately keeps the baseline editor's interaction model focused on keyboard input. Mouse support is explicitly outside the initial scope.

That decision remains reasonable.

Adding mouse support would require additional work in:

- terminal capability handling,
- input parsing,
- coordinate conversion,
- window hit testing,
- selection behavior,
- terminal-state management,
- and testing.

None of this is required for the core text-editing architecture.

The existing architecture is therefore sufficient to **accommodate** mouse support later without requiring mouse support in the baseline implementation.

## The Architectural Lesson

Mouse input demonstrates another important principle:

> **An additional physical input device does not necessarily require a new application architecture.**

The terminal emulator translates the physical mouse interaction into terminal input.

The Input Parser translates the terminal representation into an abstract event.

The Input Engine maps that event to an editor command.

The Command Engine changes the Editor State.

The architecture therefore remains:

```text
Physical Input
      │
      ▼
Terminal Protocol
      │
      ▼
Input Parser
      │
      ▼
Abstract Event
      │
      ▼
Input Engine
      │
      ▼
Command
      │
      ▼
Editor State
```

## Summary

Mouse support adds another source of input events to the existing pipeline rather than changing the fundamental input architecture.

The terminal emulator converts physical mouse interaction into terminal-level sequences; the Input Parser converts those into abstract events; the Input Engine maps events to commands; and the Command Engine changes the Editor State.

This preserves the separation between input events and commands, so the same command can be triggered by a keyboard or a mouse without changing its implementation. Mouse support is also an optional negotiated capability that should degrade gracefully when unavailable.

In the next section, we will examine the most important problem mouse interaction introduces: converting terminal coordinates into the editor's window, viewport, and document coordinate spaces.
