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

## Console

**Definition**

A hardware device or software application that provides a text-based interface to a computer system. In modern usage, "console" typically refers to the physical terminal hardware, while "terminal emulator" refers to the software that mimics it.

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

# V

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

- ASCII
- UTF-8
- Unicode
- UTF-8 Code Point
- UTF-8 Code Unit
- File Descriptor
- Shell
- PTY
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
