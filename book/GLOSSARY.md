# GLOSSARY.md

**Project:** Building a Terminal Text Editor  
**Version:** 1.0  
**Status:** Living Document

# Purpose

This glossary defines the canonical terminology used throughout the book.

It exists to ensure terminology remains consistent across every chapter.

If a term appears here, every chapter should use this definition and terminology unless explicitly stated otherwise.

This glossary is intentionally incomplete at the beginning of the project and will grow as new concepts are introduced.

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

# A

## Alternate Screen Buffer

**Definition**

A secondary, initially empty screen buffer that full-screen applications can switch into, allowing them to draw their interface without overwriting the shell's output in the normal screen buffer. Switching back restores the shell content exactly as it was.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Screen Buffer
- ANSI Escape Sequence
- Normal Screen Buffer

## ANSI Escape Sequence

**Definition**

A sequence of characters beginning with the Escape (ESC) character that instructs a terminal emulator to perform actions such as moving the cursor, clearing the screen, changing colors, or modifying display attributes.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Terminal Emulator
- Cursor
  - Raw Mode

## Application State

**Definition**

The collection of information that describes the current condition of a program at any given moment, including everything the program needs to know in order to continue operating correctly. Unlike data stored on disk, application state exists only while the program is executing and is continuously updated as the user interacts with the application.

**First Introduced**

Volume II, Chapter 12

**Related Terms**

- Editor State
- Stateful Program
- Stateless Program
- Single Source of Truth

## Architecture (Software Architecture)

**Definition**

The high-level design of a software system that identifies components, assigns responsibilities, and defines the interfaces through which those components communicate. Architecture organizes a system before implementation begins, providing a blueprint that guides development and ensures the resulting software remains comprehensible, maintainable, and extensible.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Component
- Layered Architecture
- Separation of Concerns
- Architectural Decision Record (ADR)

## Architectural Decision Record (ADR)

**Definition**

A documented design choice that captures a significant architectural decision, including the reasoning behind it, the alternatives that were considered, and the trade-offs that were accepted. ADRs preserve the context of architectural decisions so that future developers understand not only what was chosen but also why it was chosen and when the decision should be revisited.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Architecture (Software Architecture)
- Constraint

## ASCII

**Definition**

The American Standard Code for Information Interchange, a character encoding that assigns numeric values to English letters, digits, punctuation, and control characters. ASCII was one of the first widely adopted character encoding standards and remains the foundation of modern encodings such as UTF-8.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Character Encoding
- Unicode
- UTF-8

## Assumption

**Definition**

A condition that the software expects to be true about its operating environment rather than a design decision or a requirement. Assumptions allow the design to be simplified by accepting certain environmental conditions without repeatedly verifying them. Unlike guarantees, assumptions may occasionally be violated, and well-designed software handles such violations gracefully.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Constraint
- Functional Requirement
- Non-Functional Requirement (NFR)

# B

## Buffer

**Definition**

A temporary storage area that accumulates data before it is transmitted or processed. Buffering improves efficiency by reducing the number of communication operations between components, such as between a program and the operating system.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Standard Output
- Kernel

## Build System

**Definition**

A collection of tools and configuration files that automate the process of transforming source code into an executable program. The build system coordinates compilation of individual source files into object files, manages header dependencies, and invokes the linker to produce the final executable. It also supports incremental builds by recompiling only the files affected by recent changes, eliminating the need to manually compile each source file.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Object File
- Linker
- Incremental Build
- CMake

# C

## CMake

**Definition**

A cross-platform build system adopted for this project that generates native build files (such as Makefiles or Ninja files) from a single project description. CMake supports Linux, macOS, Windows, and embedded platforms, scales from small educational projects to large production systems, and integrates with modern IDEs. The project's `CMakeLists.txt` file describes source files, compiler options, and build targets.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Build System
- Object File
- Incremental Build

## Circular Dependency

**Definition**

A situation where two or more modules directly or indirectly depend on one another, forming a cycle in the dependency graph. Circular dependencies make a system harder to understand, test, and modify because a change in any module can cascade through the cycle in unexpected ways. In the editor's architecture, circular dependencies are prohibited — dependencies flow in one direction toward the Core Layer.

**First Introduced**

Volume II, Chapter 11

**Related Terms**

- Low Coupling
- Layered Architecture
- Module
- One-Way Dependencies

## Canonical Mode

**Definition**

A terminal input mode where user input is buffered by the operating system until the Enter key is pressed.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Raw Mode
- Terminal
- termios

## Character Cell

**Definition**

A fixed area of the screen reserved for displaying one character. The terminal emulator maintains a grid of character cells, each containing information such as the character to display, its foreground and background colors, and display attributes.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Monospaced Font
- Font
- Terminal Emulator

## Character Encoding

**Definition**

A system that defines the mapping between characters and the numeric values computers store and transmit. Character encodings enable software to exchange textual information reliably across different systems.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- ASCII
- Unicode
- UTF-8

## Code Point

**Definition**

A unique numeric identifier assigned to each character in the Unicode standard, written in the form `U+0041`. A code point identifies which character is intended but does not specify how it is stored as bytes — that is the role of a character encoding such as UTF-8.

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- Unicode
- UTF-8
- Character Encoding

## Coding Standard

**Definition**

A consistent set of guidelines for writing source code that improves readability, reduces misunderstandings, and makes the entire codebase feel as though it was written by a single developer. Coding standards cover naming conventions, function size, use of `const`, avoidance of magic numbers, formatting style, and commenting practices—focusing on *why* rather than *what*.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Naming Convention
- Information Hiding
- Project Structure

## Command

**Definition**

An instruction expressing an editing operation the editor should perform — such as inserting a character, moving the cursor, saving the document, or resizing the viewport. Unlike an event, which describes what happened, a command describes what the editor should do in response to an event. The mapping from events to commands is performed by the Command Engine.

**First Introduced**

Volume II, Chapter 9

**Related Terms**

- Event
- Event Loop
- Command Engine
- Editor State

## Command Engine

**Definition**

The component responsible for executing editor operations in response to parsed key events. The Command Engine determines which editing operation should be performed (such as inserting a character, deleting text, moving the cursor, or saving the document) and coordinates the resulting state changes through the Editor State.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Editor State
- Input Parser
  - File Manager

## Composition

**Definition**

An architectural pattern where complex objects are built by combining smaller, specialized components — each with its own clearly defined responsibility — rather than as a single monolithic structure. In the editor, the EditorState is composed of the Document, Cursor, Viewport, Screen, and Configuration, each contributing a distinct piece of the application's runtime state.

**First Introduced**

Volume II, Chapter 12

**Related Terms**

- Editor State
- Separation of Concerns
- Module

## Common Module

**Definition**

A foundational module that provides shared type definitions, constants, enumerations, and utility macros needed by multiple independent feature modules. The common module (typically `common/` with a `common.h` header) eliminates duplication and reduces unnecessary inter-module dependencies by serving as shared infrastructure rather than shared behavior. It must remain carefully scoped to avoid becoming a dumping ground for unrelated code.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Module
- Header File
- Information Hiding
- Separation of Concerns

## Console

**Definition**

Originally, a physical hardware device connected directly to a computer for system control. In modern usage, the term is sometimes used interchangeably with "terminal emulator," though this book distinguishes them: a console is the historical hardware device, while the terminal emulator is the software application that provides a text-based interface on a modern desktop.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Terminal Emulator
- Terminal

## Control Character

**Definition**

A character generated by holding the Control key and pressing another key, such as Ctrl+C or Ctrl+Z. In canonical mode, the terminal interprets control characters and maps them to signals (e.g., SIGINT, SIGTSTP) or special operations (e.g., end-of-file with Ctrl+D). In raw mode, control characters are delivered to the application without interpretation.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Canonical Mode
- Raw Mode
- Signal
- termios

## Control Sequence

**Definition**

A sequence of bytes sent to a terminal that is interpreted as a command rather than displayed as text. Control sequences include ANSI escape sequences and other special codes.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- ANSI Escape Sequence
- Terminal Emulator

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

## CSI (Control Sequence Introducer)

**Definition**

The two-character prefix `ESC [` that marks the beginning of a structured ANSI escape sequence. Most terminal commands used by text editors are CSI sequences.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- ANSI Escape Sequence
- SGR
- Private Mode Sequence

## Constraint

**Definition**

A rule or limitation that restricts the range of possible design and implementation choices. Unlike requirements, which describe what the software should accomplish, constraints define the boundaries within which development must occur—such as the programming language, target platform, user interface type, and concurrency model. Well-chosen constraints simplify design by eliminating incompatible solutions.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Assumption
- Functional Requirement
- Project Scope

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

**Notes**

Architectural overview: Volume II, Chapter 8. Implementation: Volume III, Chapter 14.

# D

## Data Flow

**Definition**

The predictable path that information follows through a software system as it responds to user actions. In the editor, data flow describes how a key press moves from the keyboard through the Input Handler, Input Parser, Command Engine, and Editor State, and finally to the Renderer and Terminal Display.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Architecture (Software Architecture)
- Editor State
  - Event Loop

## Dirty Flag

**Definition**

A boolean value that records whether a document has been modified since it was last saved. The dirty flag is typically false when the document matches the file on disk and true when unsaved changes exist, allowing the editor to warn the user before exiting and to avoid unnecessary save operations.

**First Introduced**

Volume II, Chapter 12

**Related Terms**

- Document
- Editor State
- File Manager

## Document

**Definition**

The editable contents of a file currently loaded into memory.

The document is represented internally by the Text Buffer.

**First Introduced**

Volume III, Chapter 12

**Related Terms**

- Text Buffer
- File Manager

**Notes**

Architectural overview: Volume II, Chapter 8. Implementation: Volume III, Chapter 12.

## Echo

**Definition**

A terminal behavior, enabled by default in canonical mode, where each character typed by the user is immediately displayed on the screen by the terminal driver. When echo is disabled in raw mode, the application becomes responsible for determining what, if anything, appears on screen in response to keyboard input.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Canonical Mode
- Raw Mode
- termios

# E

## Escape Character (ESC)

**Definition**

The ASCII control character at value 27 (0x1B) that signals the bytes following it should be interpreted as a command sequence rather than as text. ESC is the first byte of every ANSI escape sequence, used both for display control (applications sending commands to the terminal) and special-key representation (terminals sending key-press data to applications).

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- ANSI Escape Sequence
- CSI
- Control Character
- ASCII

## Editor State

**Definition**

The complete in-memory state of the editor, including the document, cursor, viewport, configuration, and other runtime information.

**First Introduced**

Volume III, Chapter 12

**Related Terms**

- Text Buffer
- Renderer
- Cursor

**Notes**

Architectural overview: Volume II, Chapter 8. Implementation: Volume III, Chapter 12.

## Event

**Definition**

Any occurrence that the application recognizes as something requiring a response. Events may originate from the user (keyboard), the operating system (terminal resize, startup, shutdown), or the application itself (cursor moved, document modified). Events are processed through a unified pipeline regardless of their source, and a single event often triggers a cascade of additional events.

**First Introduced**

Volume II, Chapter 9

**Related Terms**

- Command
- Event Loop
- Event-Driven Architecture
- Data Flow
- Editor State

## Event-Driven Architecture

**Definition**

A software system whose behavior is determined by events rather than a fixed sequence of operations. The application spends most of its lifetime waiting for events, processing each one, updating its internal state, and rendering the display — then returning to wait for the next event. This model is a natural fit for interactive applications such as text editors, where the program has nothing to do until the user presses a key or the environment changes.

**First Introduced**

Volume II, Chapter 9

**Related Terms**

- Event
- Event Loop
- Architecture (Software Architecture)
- Data Flow
- Single Source of Truth

## Event Loop

**Definition**

The continuous execution cycle that repeatedly reads input, updates application state, and renders the display until the editor exits.

**First Introduced**

Volume II, Chapter 9

**Related Terms**

- Renderer
- Input Handler

# F

## Function Key

**Definition**

A key on the keyboard labelled F1 through F12 (sometimes extending further) that has no corresponding ASCII character. Terminals transmit function-key presses as escape sequences (e.g., `ESC [ 15 ~` for F5), and applications bind them to commands such as opening menus or triggering specialised operations.

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- ANSI Escape Sequence
- Escape Character (ESC)
- Modifier Key

## Font

**Definition**

A set of glyphs that defines the visual appearance of characters. A font specifies details such as shape, size, weight, spacing, and style, and is used by the terminal emulator to determine how characters should be rendered on the screen.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Glyph
- Monospaced Font
- Terminal Emulator

## Feature Creep

**Definition**

The uncontrolled expansion of a project's scope through the continual addition of new capabilities without evaluating whether they belong within the original objectives. Feature creep increases development time, complicates testing, and often results in software that is more complex than originally intended. A well-defined project scope and success criteria help prevent feature creep.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Project Scope
- Success Criteria
- Minimum Viable Editor (MVE)

## File Manager

**Definition**

The component responsible for loading documents from disk into memory and saving the current document back to disk. The File Manager reads and writes files through operating system interfaces and updates the Editor State to reflect the result of file operations, such as clearing the Dirty Flag after a successful save.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Document
- Editor State
- Command Engine
- Dirty Flag

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

## Frame

**Definition**

A single rendered representation of the editor displayed on the terminal.

Each iteration of the event loop typically produces one frame.

**First Introduced**

Volume IV, Chapter 17

**Related Terms**

- Renderer
- Event Loop

**Notes**

Architectural overview: Volume II, Chapter 8. Implementation: Volume IV, Chapter 17.

## Full Redraw

**Definition**

The rendering strategy of regenerating the entire visible interface from the current editor state on every frame, rather than incrementally updating only the regions of the screen that changed. This avoids the complexity of tracking differences between frames.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Rendering Cycle
- Frame
- Renderer

## Functional Requirement

**Definition**

A specific behavior or service that the software must provide, describing what the system should do from the user's perspective rather than how it should be implemented. Functional requirements are written in testable terms, often using the word "shall" to indicate mandatory behavior, and are typically assigned unique identifiers (e.g., FR-001) for traceability throughout the project.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Non-Functional Requirement (NFR)
- Software Requirements Specification (SRS)
- Use Case
- Requirements Traceability

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

## Graceful Degradation

**Definition**

The design philosophy where an application uses advanced terminal features when available but falls back to simpler, widely supported alternatives when they are not, ensuring the application remains usable on all terminals.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Terminfo
- Terminal Emulator
- Capability Database

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

## Header File

**Definition**

A file (`.h`) that declares the public interface of a module—including type definitions, function prototypes, constants, and enumerations—that other modules may use. Header files serve as a contract between modules: they describe what services are available and what each function requires, while deliberately excluding implementation details. Each header is protected by an include guard to prevent multiple inclusion during compilation.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Source File
- Module
- Include Guard
- Forward Declaration
- Information Hiding

# I

## Include Guard

**Definition**

A preprocessor construct (`#ifndef`, `#define`, `#endif`) placed at the top and bottom of a header file that prevents its contents from being processed more than once during compilation of a single translation unit. Without include guards, a header included multiple times—directly or transitively—would result in duplicate declaration errors.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Header File
- Preprocessor
- Forward Declaration

## Incremental Rendering

**Definition**

A rendering strategy where only the portions of the screen that have changed since the previous frame are updated, rather than redrawing the entire display. Incremental rendering requires the renderer to track which regions are "dirty" and compute the minimal set of updates needed. While more efficient in terms of terminal output than a full redraw, it introduces additional complexity in tracking state changes and ensuring the display never becomes inconsistent.

**First Introduced**

Volume II, Chapter 11

**Related Terms**

- Full Redraw
- Rendering Cycle
- Frame
- Renderer

## Input Handler

**Definition**

The subsystem responsible for reading keyboard input and translating it into editor commands.

It does not directly modify the document.

**First Introduced**

Volume III, Chapter 15

**Related Terms**

- Input Parser
- Command Engine
- Event Loop

**Notes**

Architectural overview: Volume II, Chapter 8. Implementation: Volume III, Chapter 15.

The Input Handler is the broader subsystem. The Input Parser (introduced in Volume I, Chapter 6) is the specific byte-to-event translation layer within that subsystem.

## Input Parser

**Definition**

The component within the Input Handler responsible for translating a raw byte stream from the terminal into abstract key events. The parser examines incoming bytes, recognizes patterns such as escape sequences and multi-byte UTF-8 characters, and produces structured events (e.g., `KEY_ARROW_UP`, `KEY_CTRL_S`) that the rest of the editor can consume without knowledge of terminal protocols.

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- Input Handler
- Input Parsing
- Escape Character (ESC)
- Lookahead

**Notes**

The Input Handler (introduced in Volume III, Chapter 15) is the broader subsystem responsible for reading keyboard input and translating it into editor commands. The Input Parser is the specific byte-to-event translation layer within that subsystem.

## Information Hiding

**Definition**

A design principle where a module exposes only the information that other modules genuinely require while concealing how it accomplishes its work internally. In C projects, information hiding is achieved by declaring public types and function prototypes in header files while keeping implementation details, private helper functions, and internal data structures confined to source files—often using the `static` keyword. Information hiding reduces coupling and allows implementations to change without affecting dependents.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Module
- Header File
- Source File
- `static` (Internal Linkage)
- Separation of Concerns

## Instruction Cycle

**Definition**

The continuous process by which a CPU executes programs, consisting of three stages: fetch the next instruction from memory, decode the instruction to determine what operation to perform, and execute the operation. Also known as the fetch-decode-execute cycle.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Program Counter
- Machine Instruction

# K

## Key

**Definition**

A physical or logical button on a keyboard. Keys are distinct from characters: a single key may produce multiple bytes (e.g., an arrow key generates an escape sequence), multiple keys may produce a single character (e.g., Shift+A produces 'A'), and some keys produce no characters at all (e.g., modifier keys pressed alone).

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- Character Encoding
- Modifier Key
- Escape Character (ESC)

# L

## Layered Architecture

**Definition**

An architectural pattern that organizes software components into distinct levels of responsibility, where each layer has a specific purpose and dependencies flow in a well-defined direction. In the editor, the layers are the Application Layer (command processing), the Core Layer (editor state and data), and the Platform Layer (terminal, file, and input services).

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Architecture (Software Architecture)
- Low Coupling
- Module

## Linker

**Definition**

The tool that combines multiple object files and libraries into a single executable program. The linker resolves symbol references (function calls and variable accesses) between separately compiled modules by matching each reference to its definition, producing a complete program that the operating system can load and execute.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Object File
- Build System
- Compilation

## Line Buffering

**Definition**

A terminal behavior, enabled by default in canonical mode, where keyboard input is accumulated into an internal buffer and delivered to the application only when the user presses Enter. Line buffering allows the user to edit their input with Backspace before submission. In raw mode, line buffering is disabled so that every key press reaches the application immediately.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Canonical Mode
- Raw Mode
- termios
- Echo

## Lookahead

**Definition**

A parsing technique where the parser temporarily inspects upcoming bytes before committing to an interpretation of the current input. In terminal input, lookahead is essential because a standalone Escape key press, an Alt-modified key, and the beginning of an escape sequence all start with the same ESC byte (0x1B). The parser must read enough subsequent bytes to disambiguate.

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- Input Parser
- Escape Character (ESC)
- Input Parsing

## Low Coupling

**Definition**

A design principle where modules have few and well-defined dependencies on one another. Systems with low coupling are easier to understand, test, debug, and extend because changes to one module are less likely to affect others. The opposite is tight coupling, where many modules are interconnected and a change in one can cascade through the system.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Dependency
- High Cohesion
- Layered Architecture

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

## Minimum Viable Editor (MVE)

**Definition**

The minimal set of features that our terminal text editor must provide to be considered a functional and usable application. The MVE includes document management, text editing, cursor management, screen rendering, file operations, navigation, user interaction, and document state tracking, while intentionally excluding advanced features such as syntax highlighting, undo, plugins, and multiple documents.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Project Scope
- Functional Requirement
- Success Criteria
- Feature Creep

## Main Memory (RAM)

**Definition**

The computer's primary working area that stores both the instructions the CPU executes and the data those instructions manipulate. Main memory is volatile, meaning its contents are lost when power is removed.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Register
- Process

## Modifier Key

**Definition**

A key (Shift, Ctrl, Alt, Meta, Super) that does not produce a character on its own but alters the meaning of another key pressed simultaneously. In a terminal environment: Shift typically affects the resulting character before it reaches the application; Ctrl produces ASCII control characters; Alt often prefixes the key's normal bytes with an ESC byte; Meta and Super may be intercepted by the operating system and never reach the terminal application.

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- Key
- Control Character
- Escape Character (ESC)
- termios

## Module

**Definition**

The fundamental organizational unit in a software project, consisting of a header file that declares the public interface and a source file that contains the implementation. Each module focuses on a single architectural responsibility—such as rendering, input handling, or document management—and communicates with other modules only through its declared public interface. Modules encapsulate implementation details and reduce coupling across the codebase.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Header File
- Source File
- Information Hiding
- Separation of Concerns
- Module Interface

## Monospaced Font

**Definition**

A font in which every character occupies the same amount of horizontal space. Monospaced fonts are used by terminal emulators to ensure that text columns remain aligned, making cursor positioning and layout predictable.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Font
- Character Cell
- Terminal Emulator

# N

## Non-Functional Requirement (NFR)

**Definition**

A requirement that specifies the quality attributes or characteristics the software should possess rather than the specific features it provides. Non-functional requirements address how well the system performs (e.g., performance, reliability, usability, maintainability, portability) rather than what specific behaviors it exhibits. Unlike functional requirements, non-functional requirements often influence the design of the entire system.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Functional Requirement
- Software Requirements Specification (SRS)
- Constraint

# O

## Object File

**Definition**

An intermediate file produced by compiling a single source file into machine code before it is linked into a final executable. Object files contain compiled code and references to symbols (functions and variables) whose addresses will be resolved during linking. They allow modular compilation so that only changed files need to be recompiled.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Linker
- Build System
- Source File
- Incremental Build

# P

## Project Structure

**Definition**

The physical organization of a software project's source code into directories and files that mirrors the system architecture. Project structure determines where each architectural component lives in the codebase, how header and source files are organized, and how the build system locates and compiles the source tree. A well-designed project structure makes the architecture visible without reading a single line of implementation.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Architecture (Software Architecture)
- Module
- Header File
- Source File
- Build System

## Private Mode Sequence

**Definition**

An escape sequence beginning with `ESC[?` that controls terminal behaviour modes—such as cursor visibility, alternate screen switching, mouse reporting, and bracketed paste—rather than cursor position or text formatting. These originate from DEC VT-series terminals.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- ANSI Escape Sequence
- CSI
- Alternate Screen Buffer

## Process

**Definition**

An instance of a running program managed by the operating system. Each process has its own address space, file descriptors, and execution context.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Operating System
- File Descriptor

## Process ID (PID)

**Definition**

A unique numeric identifier assigned by the operating system to each running process. The operating system uses the PID to track, manage, and communicate with individual processes.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Process
- Operating System

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

## Program Counter (PC)

**Definition**

A special register within the CPU that stores the memory address of the next instruction to be executed. The program counter is updated after each instruction, enabling sequential execution as well as jumps, loops, and function calls.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Instruction Cycle
- Machine Instruction

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

# R

## Raw Mode

**Definition**

A terminal operating mode where input is delivered immediately to the application without line buffering or automatic processing by the terminal.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Canonical Mode
- termios

## Renderer

**Definition**

The subsystem responsible for generating the visual representation of the editor and drawing it to the terminal.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Frame
- Viewport

**Notes**

Architectural overview: Volume II, Chapter 8. Implementation: Volume IV, Chapter 19.

## Rendering Cycle

**Definition**

The fixed sequence of operations performed each time the terminal display is updated: hide the cursor, move the cursor home, clear the screen, draw the entire interface, position the cursor at the editing location, and show the cursor again.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Full Redraw
- Frame
- Renderer
- ANSI Escape Sequence

## Register

**Definition**

A tiny storage location built directly into the CPU used to hold temporary data during instruction execution. Registers are extremely fast but limited in number and capacity.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Main Memory
- Instruction Cycle

## Requirements Traceability

**Definition**

A professional practice in which every architectural component, test case, and implementation feature can be traced back to one or more requirement identifiers. Traceability ensures that every part of the system exists for a defined purpose and helps prevent unnecessary complexity, making it easier to manage changes as a project evolves.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Functional Requirement
- Software Requirements Specification (SRS)
- Non-Functional Requirement (NFR)

# S

## SGR (Select Graphic Rendition)

**Definition**

A family of ANSI escape sequences identified by the `m` final command character that control the visual appearance of text—including bold, underline, reverse video, and colors—rather than its position on screen.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- ANSI Escape Sequence
- CSI

## Screen Buffer

**Definition**

An in-memory representation of the terminal's displayed content. Modern terminal emulators maintain two screen buffers—the normal screen buffer and the alternate screen buffer—only one of which is visible at a time.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Alternate Screen Buffer
- Terminal Emulator
- ANSI Escape Sequence

## Separation of Concerns

**Definition**

A design principle that states each module in a software system should focus on a single, well-defined responsibility and avoid taking on tasks that belong elsewhere. Separation of concerns improves maintainability, readability, and scalability by ensuring that changes to one aspect of the system do not cascade through unrelated code. In a well-organized project, the directory structure and module boundaries reflect this principle.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Module
- Information Hiding
- Architecture (Software Architecture)
- Low Coupling

## Scrollback Buffer

**Definition**

A buffer maintained by the terminal emulator that stores output that has scrolled beyond the visible portion of the window. The user can view this history by scrolling upward, but the application itself is not involved in retrieving or rendering scrollback content.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- Terminal Emulator
- Viewport

## Shell

**Definition**

A command-line interpreter that reads user commands and launches programs. The shell itself does not render output—it starts processes and communicates with them through standard streams, while the terminal emulator handles display.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Terminal Emulator
- Process
- Standard Input

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

## Single Source of Truth

**Definition**

An architectural principle where every piece of application data has exactly one authoritative location. In the editor, the Editor State serves as the single source of truth for the document, cursor position, viewport, and all other runtime state. All components read from and modify this single state rather than maintaining independent copies, which prevents inconsistency and simplifies reasoning about the system.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Editor State
- State Ownership
- Data Flow

## Software Requirements Specification (SRS)

**Definition**

A structured document that defines what a software system must accomplish and the conditions under which it must operate. An SRS focuses on what the software should do rather than how it will be implemented, describing functional requirements, non-functional requirements, constraints, assumptions, use cases, and success criteria. It serves as a blueprint for the project and a shared reference for all stakeholders.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Functional Requirement
- Non-Functional Requirement (NFR)
- Constraint
- Assumption
- Use Case
- Success Criteria

## Source File

**Definition**

The implementation file (`.c`) within a module that contains the function bodies, internal helper functions, and private details of how the module performs its work. Source files translate the module's public interface—declared in the corresponding header file—into executable code. Internal functions and data marked `static` remain private to the source file and are not visible to other modules.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Header File
- Module
- Information Hiding
- Object File

## Stack

**Definition**

The region of a process's memory that stores information related to function calls, including local variables, function parameters, and return addresses. The stack grows and shrinks automatically as functions are called and return.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Heap
- Process
- Main Memory

## Stakeholder

**Definition**

Any individual or group that has an interest in a software system's development, operation, or success. Stakeholders may include end users, developers, maintainers, testers, and project managers. Each stakeholder views the software from a different perspective, and understanding these perspectives helps produce requirements that address actual needs rather than abstract capabilities.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Software Requirements Specification (SRS)
- Functional Requirement
- Use Case

## State Ownership

**Definition**

An architectural principle where each piece of data in a system has a single, clearly defined owner responsible for maintaining its correctness. In the editor, the Editor State owns the document, cursor, viewport, and all other application data. Other components may read this state but should not independently modify it — all changes flow through the owner. This ensures consistency and prevents conflicts between components.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Editor State
- Single Source of Truth
- Command Engine

## Standard Error (stderr)

**Definition**

The third standard stream provided to every process, used specifically for error messages and diagnostic output. Unlike stdout, stderr is not typically redirected when normal output is sent to a file, ensuring errors remain visible to the user.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Standard Input
- Standard Output
- Process

## Standard Input (stdin)

**Definition**

The default input stream from which a program reads data, typically connected to the keyboard in a terminal environment.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Standard Output
- Terminal

## Standard Output (stdout)

**Definition**

The default output stream to which a program writes data, typically connected to the terminal in a terminal environment.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Standard Input
  - Terminal

## Stateful Program

**Definition**

A program that remains active and continuously responds to new events by updating its internal state. A stateful program's future behavior depends on its current condition — the same input can produce different results when the program's state differs. A text editor is a stateful program because every editing operation depends on the current document, cursor position, viewport, and other runtime information.

**First Introduced**

Volume II, Chapter 12

**Related Terms**

- Application State
- Stateless Program
- Editor State

## Stateless Program

**Definition**

A program that performs a task using the information it receives, produces a result, and terminates without needing to remember anything afterward. Each execution is independent of previous runs and starts from the same initial state. Command-line utilities that take input, produce output, and exit are typically stateless.

**First Introduced**

Volume II, Chapter 12

**Related Terms**

- Application State
- Stateful Program

## Success Criteria

**Definition**

Measurable conditions that determine whether a software project has achieved its intended goals. Success criteria evaluate the system as a whole rather than describing individual behaviors, providing an objective endpoint for development and ensuring that the implementation remains aligned with the project's scope.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Project Scope
- Functional Requirement
- Minimum Viable Editor (MVE)
- Feature Creep

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

## Terminfo

**Definition**

A capability database on Unix-like systems that provides terminal-specific control sequences. Applications query terminfo for the correct escape codes for the current terminal type rather than hard-coding sequences, greatly improving portability across different terminal emulators.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Terminal Emulator
- ANSI Escape Sequence
- Graceful Degradation

## termios

**Definition**

A POSIX interface that allows applications to inspect and modify terminal configuration settings. Through `struct termios` and functions such as `tcgetattr()` and `tcsetattr()`, a program can enable or disable features including canonical mode, echo, signal generation, and input/output processing. The interface is the mechanism by which applications enter and exit raw mode.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Canonical Mode
- Raw Mode
- Line Buffering
- Echo
- Control Character

## Text Buffer

**Definition**

The in-memory data structure that stores the editable contents of the current document.

All editing operations modify the Text Buffer rather than the file on disk.

**First Introduced**

Volume III, Chapter 13

**Related Terms**

- Document
- Cursor

## TTY

**Definition**

Originally a teletypewriter, now commonly used to refer to a terminal device managed by the operating system.

**First Introduced**

Volume I, Chapter 3

**Related Terms**

- PTY
- Terminal

# U

## Use Case

**Definition**

A description of a sequence of interactions between a user and a software system to accomplish a specific goal. Unlike individual requirements, which describe isolated pieces of functionality, use cases describe complete tasks from the user's perspective. Each use case identifies an actor, a goal, and the typical flow of interactions required to achieve that goal.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Functional Requirement
- Stakeholder
- Software Requirements Specification (SRS)

## Unicode

**Definition**

A universal character standard that assigns a unique numeric identifier—called a code point—to virtually every character used in modern writing systems, including alphabets, ideographs, mathematical symbols, and emoji.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- UTF-8
- ASCII
- Character Encoding

## UTF-8

**Definition**

The dominant character encoding used on modern Unix-like systems and the Internet. UTF-8 uses a variable-length encoding where common ASCII characters occupy one byte while other characters may require multiple bytes. It is fully backward compatible with ASCII.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- Unicode
- ASCII
- Character Encoding

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

## Viewport

**Definition**

The visible portion of the document currently displayed on the screen.

The viewport changes as the user scrolls through the document.

**First Introduced**

Volume IV, Chapter 20

**Related Terms**

- Renderer
- Cursor

**Notes**

Architectural overview: Volume II, Chapter 8. Implementation: Volume IV, Chapter 20.

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

# Future Terms

The following concepts are expected to be added as the manuscript develops:

- Gap Buffer
- Piece Table
- Rope
- Status Bar
- Message Bar
- Immediate Mode Rendering
- Retained Mode Rendering
- Scroll Offset
- Horizontal Scrolling
- Vertical Scrolling
- Atomic Save
- Language Server Protocol (LSP)
- Tree-sitter
- Syntax Highlighting
- Double Buffering
- Cache Locality

# Maintenance Rules

1. Every canonical technical term should appear exactly once in this glossary.

2. Definitions should be concise and technology-agnostic where possible.

3. If a definition changes, update the glossary before modifying individual chapters.

4. Do not duplicate glossary definitions inside chapters unless necessary for comprehension.

5. Chapters should use glossary terminology consistently.

6. When introducing a new canonical term, immediately add it to this document.

7. If two terms appear to describe the same concept, prefer the glossary definition and terminology.

End of GLOSSARY.md
