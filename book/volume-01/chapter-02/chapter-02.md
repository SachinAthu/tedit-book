# Chapter Outline

## 2.1 The Journey of a Character

Begin with a high-level overview.

Example:

```
Your Program
      │
      ▼
Operating System
      │
      ▼
Pseudo Terminal (PTY)
      │
      ▼
Terminal Emulator
      │
      ▼
Graphics System
      │
      ▼
Monitor
```

Explain that the remainder of the chapter examines each stage.

---

## 2.2 The CPU

Topics:

- What a CPU is
- Instructions
- Registers
- Executing machine code
- Running one instruction at a time
- Why the CPU cannot display text itself

Diagram:

```
Memory
   │
   ▼
+---------+
|   CPU   |
+---------+
     │
 Executes
 Instructions
```

---

## 2.3 Main Memory (RAM)

Topics:

- Programs and data stored in memory
- Variables
- Stack vs heap (high-level only)
- Why text editors keep documents in memory
- Loading before editing

Diagram:

```
+--------------------+
| Program Code       |
| Stack              |
| Heap               |
| Text Buffer        |
+--------------------+
```

Include a note that deeper memory management will be discussed later.

---

## 2.4 Processes

Topics:

- Definition of a process
- Program vs process
- Process state
- Process memory
- Multiple running programs

Diagram:

```
Executable File

editor

      │ Launch

      ▼

Running Process

PID: 2451

Memory
Registers
Files
```

---

## 2.5 Standard Input and Standard Output

Introduce:

- stdin
- stdout
- stderr (briefly)

Explain why they exist.

Example:

```
Keyboard

↓

stdin

↓

Program

↓

stdout

↓

Terminal
```

Show:

```c
printf("Hello");
```

Explain that `printf()` writes to stdout—not directly to the display.

---

## 2.6 From stdout to the Terminal

Topics:

- Output stream
- Bytes
- Buffering (only conceptually)
- OS forwarding output

Diagram:

```
printf()

↓

stdout

↓

Kernel

↓

PTY

↓

Terminal Emulator
```

No PTY internals yet—that comes in Chapter 3.

---

## 2.7 Terminal Emulator

Explain:

- What a terminal emulator is
- Why it exists
- Examples
  - GNOME Terminal
  - Windows Terminal
  - Alacritty
  - Kitty
  - iTerm2

Explain that these programs interpret character streams rather than execute shell commands themselves.

Diagram:

```
Application

↓

Character Stream

↓

Terminal Emulator

↓

Rendered Window
```

---

## 2.8 Console vs Terminal

Compare:

Console

- Historical hardware
- Physical machine

Terminal

- Modern software
- Window on the desktop

Table:

| Console            | Terminal Emulator   |
| ------------------ | ------------------- |
| Physical           | Software            |
| Historical         | Modern              |
| Dedicated hardware | Regular application |

Mention that the terms are often used interchangeably.

---

## 2.9 Character Rendering

Explain:

- Fonts
- Glyphs
- Character cells
- Monospaced fonts
- Cursor positioning

Diagram:

```
+---+---+---+---+
| H | e | l | l |
+---+---+---+---+
| o |   |   |   |
+---+---+---+---+
```

Introduce the concept of a character grid.

---

## 2.10 Character Encoding

Start with ASCII:

```
'A'

↓

65

↓

01000001
```

Explain why ASCII became insufficient.

Then introduce Unicode.

Finally explain UTF-8:

- Variable-length encoding
- Backward compatible with ASCII
- Modern standard

Avoid implementation details.

---

## 2.11 Following One Character

Walk through the complete journey.

Example:

```
printf("A");
```

Flow:

```
CPU executes printf()

↓

stdout

↓

Kernel

↓

PTY

↓

Terminal Emulator

↓

Font

↓

Glyph

↓

GPU

↓

Monitor
```

This section ties together the entire chapter.

---
