# Volume I — Computing Foundations

# Chapter 4 – ANSI Escape Sequences

## Introduction

- From printing text to controlling the terminal
- The terminal as a command interpreter
- Why ANSI escape sequences are the terminal's language

---

## Learning Objectives

By the end of this chapter, readers will be able to:

- Explain what ANSI escape sequences are.
- Understand how terminals interpret escape codes.
- Move the cursor programmatically.
- Clear parts of the screen.
- Change text appearance and colors.
- Use the alternate screen buffer.
- Hide and show the cursor.
- Understand why these capabilities are essential for building a text editor.

---

# 4.1 The Problem with Plain Text

Introduce the limitation of normal output.

Example:

```c
printf("Hello");
printf("World");
```

Explain that output is strictly sequential.

Questions explored:

- Why can't we overwrite text?
- Why can't we move the cursor?
- Why can't we clear the screen?

Diagram:

```
Program

↓

stdout

↓

Hello
World
```

---

# 4.2 What Is an ANSI Escape Sequence?

Introduce the concept.

Explain:

- ASCII ESC (27)
- Escape character (`\x1b`)
- Command language for terminals

Diagram:

```
ESC

↓

Terminal switches

from

"print mode"

to

"command mode"
```

Example:

```
"\x1b[2J"
```

Break it apart character by character.

---

# 4.3 Anatomy of an Escape Sequence

Explain the structure.

Example:

```
ESC [ 31 m
```

Breakdown:

```
ESC

[

Parameters

Final Command
```

Introduce:

- CSI (Control Sequence Introducer)
- Parameters
- Final byte

Readers learn the grammar instead of memorizing commands.

---

# 4.4 Cursor Movement

Core commands.

Topics:

- Move to row/column
- Move up
- Move down
- Move left
- Move right
- Save cursor position
- Restore cursor position

ASCII diagrams showing cursor movement.

Example:

```
Hello
World
 ^
```

↓

```
Hello
     ^
World
```

---

# 4.5 Clearing the Screen

Explain:

- Clear entire screen
- Clear to end of line
- Clear current line
- Clear scrollback (briefly)

Compare:

```
Printing blank lines

vs

Actual terminal clearing
```

---

# 4.6 Text Formatting

Introduce SGR (Select Graphic Rendition).

Topics:

- Reset
- Bold
- Underline
- Reverse video
- Foreground colors
- Background colors
- 256-color mode
- True color (overview)

Explain why our editor will use only a small subset initially.

---

# 4.7 Cursor Visibility

Topics:

- Hide cursor
- Show cursor
- Why editors hide the cursor while redrawing
- Preventing visual flicker

Illustration of a redraw cycle.

---

# 4.8 Alternate Screen Buffer

One of the most important concepts for editor development.

Explain:

```
Normal Screen

↓

Enter Alternate Screen

↓

Editor Runs

↓

Leave Alternate Screen

↓

Original Shell Returns
```

Discuss:

- Why Vim and Nano don't destroy your shell output.
- Why full-screen applications use this feature.

---

# 4.9 Putting It All Together

Walk through a complete redraw.

Example flow:

```
Program

↓

Hide Cursor

↓

Move Cursor Home

↓

Clear Screen

↓

Draw Lines

↓

Move Cursor

↓

Show Cursor
```

Explain that **every frame of our editor will follow this pattern**.

---

# 4.10 Terminal Compatibility

Discuss:

- VT100
- ANSI standard
- Modern terminal emulators
- xterm compatibility
- Windows Terminal
- Why these sequences generally work everywhere

Also introduce the idea that not every terminal supports every feature.

---

# 4.11 Looking Ahead: Building Our Renderer

Bridge to Chapter 5.

Show how escape sequences fit into the rendering pipeline established in Chapter 2 and the terminal architecture from Chapter 3.

```
Editor State

↓

Renderer

↓

ANSI Escape Sequences

↓

stdout

↓

PTY

↓

Terminal Emulator

↓

Screen
```

This section connects all previous chapters and motivates the next one.

---
