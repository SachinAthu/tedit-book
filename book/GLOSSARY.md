# GLOSSARY.md

**Project:** Building a Terminal Text Editor  
**Version:** 1.0  
**Status:** Living Document

---

# Purpose

This glossary defines the canonical terminology used throughout the book.

It exists to ensure terminology remains consistent across every chapter.

If a term appears here, every chapter should use this definition and terminology unless explicitly stated otherwise.

This glossary is intentionally incomplete at the beginning of the project and will grow as new concepts are introduced.

---

# Conventions

Each entry should contain:

- Definition
- First Introduced
- Related Terms
- Notes (optional)

Example:

## Term

**Definition**

...

**First Introduced**

Volume X, Chapter Y

**Related Terms**

- ...
- ...

---

# A

## ANSI Escape Sequence

**Definition**

A sequence of characters beginning with the Escape (ESC) character that instructs a terminal emulator to perform actions such as moving the cursor, clearing the screen, changing colors, or modifying display attributes.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Terminal Emulator
- Cursor
- Raw Mode

---

## ASCII

**Definition**

The American Standard Code for Information Interchange, a character encoding that assigns numeric values to English letters, digits, punctuation, and control characters. ASCII was one of the first widely adopted character encoding standards and remains the foundation of modern encodings such as UTF-8.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Character Encoding
- Unicode
- UTF-8

---

# B

## Buffer

**Definition**

A temporary storage area that accumulates data before it is transmitted or processed. Buffering improves efficiency by reducing the number of communication operations between components, such as between a program and the operating system.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Standard Output
- Kernel

---

# C

## Canonical Mode

**Definition**

A terminal input mode where user input is buffered by the operating system until the Enter key is pressed.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Raw Mode
- Terminal
- termios

---

## Character Cell

**Definition**

A fixed area of the screen reserved for displaying one character. The terminal emulator maintains a grid of character cells, each containing information such as the character to display, its foreground and background colors, and display attributes.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Monospaced Font
- Font
- Terminal Emulator

---

## Character Encoding

**Definition**

A system that defines the mapping between characters and the numeric values computers store and transmit. Character encodings enable software to exchange textual information reliably across different systems.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- ASCII
- Unicode
- UTF-8

---

## Console

**Definition**

Originally, a physical hardware device connected directly to a computer for system control. In modern usage, the term is sometimes used interchangeably with "terminal emulator," though this book distinguishes them: a console is the historical hardware device, while the terminal emulator is the software application that provides a text-based interface on a modern desktop.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Terminal Emulator
- Terminal

---

## Control Sequence

**Definition**

A sequence of bytes sent to a terminal that is interpreted as a command rather than displayed as text. Control sequences include ANSI escape sequences and other special codes.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- ANSI Escape Sequence
- Terminal Emulator

---

## Controlling Terminal

**Definition**

The terminal device associated with an interactive process or process group. It serves as the primary communication channel between the user and the process, delivering keyboard input, routing output, and generating terminal signals such as those triggered by Ctrl+C and Ctrl+Z.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- Terminal Session
- Process Group
- PTY
- Signal

---

## CPU (Central Processing Unit)

**Definition**

The component responsible for executing instructions and performing computations. The CPU fetches, decodes, and executes machine instructions in a continuous cycle while using registers to hold temporary data and the program counter to track the next instruction.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Machine Instruction
- Instruction Cycle
- Register
- Program Counter
- Main Memory

---

## Cursor

**Definition**

The logical position within the document where the next editing operation occurs.

The cursor has both a document position and a screen position, although these may differ when scrolling is involved.

**First Introduced**

Volume III, Chapter 14

**Related Terms**

- Viewport
- Renderer
- Text Buffer

---

# D

## Document

**Definition**

The editable contents of a file currently loaded into memory.

The document is represented internally by the Text Buffer.

**First Introduced**

Volume III, Chapter 12

**Related Terms**

- Text Buffer
- File Manager

---

# E

## Editor State

**Definition**

The complete in-memory state of the editor, including the document, cursor, viewport, configuration, and other runtime information.

**First Introduced**

Volume III, Chapter 12

**Related Terms**

- Text Buffer
- Renderer
- Cursor

---

## Event Loop

**Definition**

The continuous execution cycle that repeatedly reads input, updates application state, and renders the display until the editor exits.

**First Introduced**

Volume II, Chapter 9

**Related Terms**

- Renderer
- Input Handler

---

# F

## Font

**Definition**

A set of glyphs that defines the visual appearance of characters. A font specifies details such as shape, size, weight, spacing, and style, and is used by the terminal emulator to determine how characters should be rendered on the screen.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Glyph
- Monospaced Font
- Terminal Emulator

---

## File Descriptor

**Definition**

A small non-negative integer that identifies an open file or I/O resource within a process. Despite the name, a file descriptor can refer to a regular file, a terminal, a pipe, a socket, or many other input/output resources. Every process begins with three standard file descriptors: 0 (stdin), 1 (stdout), and 2 (stderr).

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- Standard Input
- Standard Output
- Standard Error
- Process
- PTY

---

## Frame

**Definition**

A single rendered representation of the editor displayed on the terminal.

Each iteration of the event loop typically produces one frame.

**First Introduced**

Volume IV, Chapter 17

**Related Terms**

- Renderer
- Event Loop

---

# G

## Glyph

**Definition**

The visual representation of a character. While a character is an abstract piece of information, a glyph is the specific shape used to display that character in a given font.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Character
- Font
- Unicode

---

# H

## Heap

**Definition**

The region of a process's memory used for dynamically allocated data. Memory on the heap is explicitly requested and released by the program during execution, typically through functions such as `malloc()` and `free()`.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Stack
- Process
- Main Memory

---

# I

## Input Handler

**Definition**

The subsystem responsible for reading keyboard input and translating it into editor commands.

It does not directly modify the document.

**First Introduced**

Volume III, Chapter 15

**Related Terms**

- Command Engine
- Event Loop

---

## Instruction Cycle

**Definition**

The continuous process by which a CPU executes programs, consisting of three stages: fetch the next instruction from memory, decode the instruction to determine what operation to perform, and execute the operation. Also known as the fetch-decode-execute cycle.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Program Counter
- Machine Instruction

---

# M

## Machine Instruction

**Definition**

A single operation that a CPU can execute, encoded as binary data. Machine instructions are the lowest-level commands understood by the processor, such as adding two numbers, copying data, or jumping to another instruction.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Instruction Cycle
- Program Counter

---

## Main Memory (RAM)

**Definition**

The computer's primary working area that stores both the instructions the CPU executes and the data those instructions manipulate. Main memory is volatile, meaning its contents are lost when power is removed.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Register
- Process

---

## Monospaced Font

**Definition**

A font in which every character occupies the same amount of horizontal space. Monospaced fonts are used by terminal emulators to ensure that text columns remain aligned, making cursor positioning and layout predictable.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Font
- Character Cell
- Terminal Emulator

---

# P

## Process

**Definition**

An instance of a running program managed by the operating system. Each process has its own address space, file descriptors, and execution context.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Operating System
- File Descriptor

---

## Process ID (PID)

**Definition**

A unique numeric identifier assigned by the operating system to each running process. The operating system uses the PID to track, manage, and communicate with individual processes.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Process
- Operating System

---

## Process Group

**Definition**

A collection of one or more related processes that the operating system manages as a single unit for the purposes of signal delivery, terminal management, and job control. Every controlling terminal has one foreground process group, which has exclusive access to keyboard input and terminal-generated signals. Other process groups may run in the background but do not receive interactive input.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- Controlling Terminal
- Foreground Process Group
- Job Control
- Signal
- Shell

---

## Program Counter (PC)

**Definition**

A special register within the CPU that stores the memory address of the next instruction to be executed. The program counter is updated after each instruction, enabling sequential execution as well as jumps, loops, and function calls.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Instruction Cycle
- Machine Instruction

---

## Pseudo Terminal (PTY)

**Definition**

A pair of virtual devices that emulate a real terminal entirely in software. The PTY consists of two connected ends: the master, controlled by the terminal emulator, and the slave, which behaves like a real terminal device for shells and applications. PTYs allow graphical terminal emulators to provide a terminal environment without dedicated hardware. Their device files typically appear under `/dev/pts/`.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- TTY
- Terminal Emulator
- Controlling Terminal
- File Descriptor
- Virtual Console

---

# R

## Raw Mode

**Definition**

A terminal operating mode where input is delivered immediately to the application without line buffering or automatic processing by the terminal.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Canonical Mode
- termios

---

## Renderer

**Definition**

The subsystem responsible for generating the visual representation of the editor and drawing it to the terminal.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Frame
- Viewport

---

## Register

**Definition**

A tiny storage location built directly into the CPU used to hold temporary data during instruction execution. Registers are extremely fast but limited in number and capacity.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Main Memory
- Instruction Cycle

---

# S

## Scrollback Buffer

**Definition**

A buffer maintained by the terminal emulator that stores output that has scrolled beyond the visible portion of the window. The user can view this history by scrolling upward, but the application itself is not involved in retrieving or rendering scrollback content.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- Terminal Emulator
- Viewport

---

## Shell

**Definition**

A command-line interpreter that reads user commands and launches programs. The shell itself does not render output—it starts processes and communicates with them through standard streams, while the terminal emulator handles display.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Terminal Emulator
- Process
- Standard Input

---

## Signal

**Definition**

A notification delivered by the operating system to a process or process group in response to an event. Terminal-generated signals are triggered by keyboard shortcuts such as Ctrl+C (SIGINT) and Ctrl+Z (SIGTSTP) and are delivered to the foreground process group. Signals allow the user to interrupt, suspend, or terminate running programs without the application needing to poll for these events.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- Controlling Terminal
- Process Group
- Foreground Process Group
- Job Control

---

## Stack

**Definition**

The region of a process's memory that stores information related to function calls, including local variables, function parameters, and return addresses. The stack grows and shrinks automatically as functions are called and return.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Heap
- Process
- Main Memory

---

## Standard Error (stderr)

**Definition**

The third standard stream provided to every process, used specifically for error messages and diagnostic output. Unlike stdout, stderr is not typically redirected when normal output is sent to a file, ensuring errors remain visible to the user.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Standard Input
- Standard Output
- Process

---

## Standard Input (stdin)

**Definition**

The default input stream from which a program reads data, typically connected to the keyboard in a terminal environment.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Standard Output
- Terminal

---

## Standard Output (stdout)

**Definition**

The default output stream to which a program writes data, typically connected to the terminal in a terminal environment.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Standard Input
- Terminal

---

# T

## Teletype (TTY)

**Definition**

An electromechanical terminal device consisting of a keyboard and a paper printer, originally developed for telegraph networks and later connected directly to computers. Teletypes printed every character onto paper as it was received, creating a permanent record of each session. Although these machines have long disappeared, Unix retained their name: the term TTY still refers to terminal devices managed by the operating system.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- TTY
- Terminal
- Video Terminal

---

## Terminal

**Definition**

The text-based interface through which users interact with command-line programs.

In this book, "terminal" generally refers to a terminal emulator unless otherwise specified.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Terminal Emulator
- TTY
- PTY

---

## Terminal Emulator

**Definition**

A graphical application that emulates the behavior of a hardware terminal by interpreting text output and ANSI escape sequences.

Examples include Windows Terminal, GNOME Terminal, iTerm2, and Alacritty.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Terminal
- ANSI Escape Sequence
- TTY

---

## Terminal Session

**Definition**

The complete interactive conversation between a user and the operating system through a terminal, encompassing the terminal emulator, the PTY, the shell, and any programs launched from that shell. A terminal session begins when the terminal emulator starts and creates a PTY, and ends when the shell exits and the terminal window closes.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- Terminal Emulator
- PTY
- Shell
- Controlling Terminal
- Process Group

---

## Text Buffer

**Definition**

The in-memory data structure that stores the editable contents of the current document.

All editing operations modify the Text Buffer rather than the file on disk.

**First Introduced**

Volume III, Chapter 13

**Related Terms**

- Document
- Cursor

---

## TTY

**Definition**

Originally a teletypewriter, now commonly used to refer to a terminal device managed by the operating system.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- PTY
- Terminal

---

# U

## Unicode

**Definition**

A universal character standard that assigns a unique numeric identifier—called a code point—to virtually every character used in modern writing systems, including alphabets, ideographs, mathematical symbols, and emoji.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- UTF-8
- ASCII
- Character Encoding

---

## UTF-8

**Definition**

The dominant character encoding used on modern Unix-like systems and the Internet. UTF-8 uses a variable-length encoding where common ASCII characters occupy one byte while other characters may require multiple bytes. It is fully backward compatible with ASCII.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Unicode
- ASCII
- Character Encoding

---

# V

## Video Terminal

**Definition**

A hardware terminal that replaced paper printers with a cathode-ray tube (CRT) display. Video terminals allowed users to move the cursor, edit text on the screen, clear parts of the display, and create full-screen interactive applications for the first time. The control sequences developed for these terminals eventually became standardized as ANSI escape sequences.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- ANSI Escape Sequence
- Terminal
- Teletype
- Terminal Emulator

---

## Viewport

**Definition**

The visible portion of the document currently displayed on the screen.

The viewport changes as the user scrolls through the document.

**First Introduced**

Volume IV, Chapter 20

**Related Terms**

- Renderer
- Cursor

---

## Virtual Console

**Definition**

A software-created terminal that provides an independent login session using the same physical keyboard and monitor as other virtual consoles. Managed directly by the Linux kernel and accessed via Ctrl+Alt+Fn shortcuts, each virtual console has its own TTY device (e.g., `/dev/tty1`) and runs independently of graphical terminal emulators.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- TTY
- PTY
- Terminal Emulator
- Console

---

# W

## Working Tree

**Definition**

The editable files currently checked out in the Git repository.

This term is relevant to the book's writing workflow rather than the editor implementation.

**First Introduced**

Project Documentation

**Related Terms**

- Git
- Repository

---

# Future Terms

The following concepts are expected to be added as the manuscript develops:

- UTF-8 Code Point
- UTF-8 Code Unit
- termios
- Gap Buffer
- Piece Table
- Rope
- Status Bar
- Message Bar
- File Manager
- Command Engine
- Immediate Mode Rendering
- Retained Mode Rendering
- Dirty Flag
- Scroll Offset
- Horizontal Scrolling
- Vertical Scrolling
- Atomic Save
- Language Server Protocol (LSP)
- Tree-sitter
- Syntax Highlighting
- Double Buffering
- Cache Locality

---

# Maintenance Rules

1. Every canonical technical term should appear exactly once in this glossary.

2. Definitions should be concise and technology-agnostic where possible.

3. If a definition changes, update the glossary before modifying individual chapters.

4. Do not duplicate glossary definitions inside chapters unless necessary for comprehension.

5. Chapters should use glossary terminology consistently.

6. When introducing a new canonical term, immediately add it to this document.

7. If two terms appear to describe the same concept, prefer the glossary definition and terminology.

---

End of GLOSSARY.md
