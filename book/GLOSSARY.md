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

## Add Buffer

**Definition**

The buffer in a Piece Table that stores text introduced during editing. Newly inserted text is appended to the Add Buffer, which therefore only grows; existing contents are never modified or removed. Because the Add Buffer is append-only, text that is later deleted from the logical document can remain stored in the buffer, and its physical size is not a reliable indicator of the current document's length.

**First Introduced**

Volume VI, Chapter 27

**Related Terms**

- Piece Table
- Original Buffer
- Piece Descriptor
- Logical Document

## Alternate Screen Buffer

**Definition**

A secondary, initially empty screen buffer that full-screen applications can switch into, allowing them to draw their interface without overwriting the shell's output in the normal screen buffer. Switching back restores the shell content exactly as it was.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Screen Buffer
- ANSI Escape Sequence
- Normal Screen Buffer

## Amortized Complexity

**Definition**

An analysis method that determines the cost of a data structure's operations by distributing the cost of occasional expensive operations across an entire sequence of operations, rather than judging a single operation in isolation. The amortized cost of an insertion can be much smaller than the cost of the occasional resize, but amortized analysis does not make the expensive operation physically faster, so it is considered alongside worst-case behavior when evaluating an interactive editor.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Worst Case
- Average Case
- Time Complexity
- Complexity Analysis

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

Volume III, Chapter 12

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

## Array

**Definition**

A data structure that stores a collection of elements in a defined order, where each element can be accessed by its position, or index. The elements occupy a contiguous region of memory, so the address of an indexed element can be calculated directly from the base address, making indexed access fast and predictable. For a text editor, a line maps naturally onto an array of characters; array-based storage is simple and provides efficient reading, but inserting or deleting elements in the middle requires moving the elements that follow to preserve the contiguous sequence.

**First Introduced**

Volume VI, Chapter 25

**Related Terms**

- Contiguous Storage
- Fixed-Size Array
- Dynamic Array
- Length
- Capacity
- Vector of Lines
- Text Buffer

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

## Atomic Replacement

**Definition**

A mechanism that replaces the contents of a file as one indivisible operation, so that the destination never exposes a deliberately partial or missing intermediate state. In our editor, the File Manager prepares the complete new version in a Temporary File and then asks the operating system to replace the destination with it using `rename()`, which is atomic only when the source and destination are on the same filesystem.

**First Introduced**

Volume V, Chapter 23

**Related Terms**

- Atomicity
- Temporary File
- Durability
- File Manager

## Atomicity

**Definition**

The property of an operation that it appears to happen as one indivisible action, with no intentionally exposed intermediate state. Atomicity describes how a change happens; it is distinct from durability, which describes whether a successful change survives a later failure such as a crash or power loss.

**First Introduced**

Volume V, Chapter 23

**Related Terms**

- Atomic Replacement
- Durability

## Average Case

**Definition**

The expected cost of an operation across a representative distribution of inputs or states. Average-case analysis depends on assumptions about which inputs are likely, which is why worst-case analysis is often easier to trust when evaluating whether a design can tolerate large inputs in an interactive application.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Best Case
- Worst Case
- Complexity Analysis
- Time Complexity

# B

## Best Case

**Definition**

The possible cost of an operation under the most favorable arrangement of its input or state. The best case shows how cheaply an operation can finish, but it does not describe how the operation normally behaves, so it is not a reliable basis for choosing a design.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Average Case
- Worst Case
- Time Complexity
- Big-O Notation

## Big-O Notation

**Definition**

A notation that describes how the resource requirements of an operation grow as its input size increases, focusing on the growth rate rather than on constant factors or exact execution time. An operation whose work grows proportionally with its input is described as `O(n)`. Big-O is a model of growth, not a measurement of speed.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Complexity Analysis
- Time Complexity
- Space Complexity
- Input Size

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

## Cache Locality

**Definition**

A measure of how efficiently a program accesses memory, based on whether its access patterns make effective use of the CPU cache. Temporal locality describes reusing the same data soon, and spatial locality describes accessing nearby data, so contiguous layouts combined with sequential access tend to perform better in practice even at the same Big-O complexity.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Temporal Locality
- Spatial Locality
- Main Memory (RAM)
- Complexity Analysis

## Capacity

**Definition**

The total number of elements that an array's allocated storage can hold, as opposed to the length, which is the number of elements currently in use. Unused capacity remains part of the allocation but not part of the sequence, letting a dynamic array accept future insertions without immediately allocating new storage. Distinguishing length from capacity is central to dynamic arrays and dynamic strings.

**First Introduced**

Volume VI, Chapter 25

**Related Terms**

- Length
- Fixed-Size Array
- Dynamic Array
- Dynamic String
- Resizing
- Array

**Notes**

A C string complicates the accounting: the allocated storage must also accommodate its null terminator, which belongs to the representation rather than to the logical text length.

## Contiguous Storage

**Definition**

A memory layout in which the elements of a sequence occupy one continuous region of memory, each element following the previous one at a predictable distance. Contiguity is what makes array indexing direct — the address of an element follows from its base address and index — and it also makes array insertion and deletion costly, because elements must move to keep the sequence continuous. A dynamic array changes the size of its storage but preserves contiguity within each allocation.

**First Introduced**

Volume VI, Chapter 25

**Related Terms**

- Array
- Main Memory (RAM)
- Cache Locality
- Dynamic Array

## Complexity Analysis

**Definition**

The study of how an operation's use of computational resources grows as its input grows. The editor analyzes each operation together with the data structure that implements it, identifying the relevant input size, the work performed, and how that work scales, because the representation of text determines which operations are cheap and which become expensive as the document grows.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Time Complexity
- Space Complexity
- Big-O Notation
- Input Size
- Amortized Complexity
- Latency

## Constant Time

**Definition**

The complexity classification of an operation that performs a bounded amount of work which does not grow with the input size, written `O(1)`. Constant time means the amount of work is unaffected by the size of the input; it does not imply that the operation is fast in absolute terms.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Big-O Notation
- Linear Time
- Quadratic Time
- Time Complexity

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

## Character Set

**Definition**

A defined collection of characters that can be represented, independent of how those characters are stored as bytes. A character set answers the question of which characters are available — for example, ASCII defines 128 characters — but does not by itself specify the byte representation; that is the role of a character encoding such as UTF-8.

**First Introduced**

Volume V, Chapter 22

**Related Terms**

- Character Encoding
- Unicode
- ASCII
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

## C String

**Definition**

In C, a sequence of characters (bytes) stored in a contiguous array and terminated by a null byte (`'\0'`). The null byte marks the end of the string, so C functions such as `strlen()` determine its length by scanning rather than by reading a stored length. Because a C `char` represents a byte, a C string is fundamentally a byte sequence: for simple ASCII text it looks like a sequence of characters, but multi-byte encodings such as UTF-8 break that equivalence. The editor's Document instead stores decoded Unicode code points and does not use C strings as its fundamental text representation.

**First Introduced**

Volume VI, Chapter 25

**Related Terms**

- Null Terminator
- Dynamic String
- Array
- Code Point
- UTF-8
- Character Encoding

## Coding Standard

**Definition**

A consistent set of guidelines for writing source code that improves readability, reduces misunderstandings, and makes the entire codebase feel as though it was written by a single developer. Coding standards cover naming conventions, function size, use of `const`, avoidance of magic numbers, formatting style, and commenting practices—focusing on _why_ rather than _what_.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Naming Convention
- Information Hiding
- Project Structure

## Command

**Definition**

An instruction expressing an editing operation the editor should perform — such as inserting a character, moving the cursor, saving the document, or resizing the viewport. Unlike an event, which describes what happened, a command describes what the editor should do in response to an event. The mapping from events to commands is performed by the Input Engine.

**First Introduced**

Volume II, Chapter 9

**Related Terms**

- Event
- Event Loop
- Input Engine
- Command Engine
- Editor State

**Notes**

Detailed treatment: Volume III, Chapter 16. A command is a value — a command type plus optional command data — not the implementation of the operation it describes.

## Command Data

**Definition**

The optional payload carried alongside a command type that provides the information a particular operation requires. For example, an `INSERT_CHARACTER` command carries the character to insert, while a movement command requires no additional data. Separating the command type from its data lets one representation support both parameterless and parameterized commands.

**First Introduced**

Volume III, Chapter 16

**Related Terms**

- Command
- Command Type
- Command Engine

## Command Dispatch

**Definition**

The mechanism by which the Command Engine selects the operation corresponding to a received command type. For the editor's small, fixed command vocabulary, dispatch is implemented as a `switch` statement over the command type that passes any command data to the selected core operation. Dispatch selects an operation but does not implement it.

**First Introduced**

Volume III, Chapter 16

**Related Terms**

- Command
- Command Type
- Command Data
- Command Engine

## Command Engine

**Definition**

The component responsible for executing editor commands. The Command Engine receives a command dispatched by the Input Engine, determines how that command should change the editor (such as inserting a character, deleting text, moving the cursor, or saving the document), and coordinates the resulting state changes through the Editor State.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Input Engine
- Command
- Editor State
- File Manager

**Notes**

Architectural overview: Volume II, Chapter 8. Detailed treatment: Volume III, Chapter 16.

## Command Type

**Definition**

The portion of a command that identifies which operation should be performed. In C, the command type is represented as an enumeration (`CommandType`) whose values — such as `COMMAND_MOVE_CURSOR_LEFT`, `COMMAND_INSERT_CHARACTER`, and `COMMAND_SAVE_DOCUMENT` — describe editor behavior rather than the keyboard input that produced it.

**First Introduced**

Volume III, Chapter 16

**Related Terms**

- Command
- Command Data
- Command Dispatch
- Command Engine

## Composition

**Definition**

An architectural pattern where complex objects are built by combining smaller, specialized components — each with its own clearly defined responsibility — rather than as a single monolithic structure. In the editor, the EditorState is composed of the Document, Cursor, Viewport, Screen Size, and Configuration, each contributing a distinct piece of the application's runtime state.

**First Introduced**

Volume III, Chapter 12

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

The logical editing position within the document, described by a row and a column, where the next editing operation will occur.

The cursor is a logical position in the Document rather than the blinking character on the terminal; the terminal cursor is only its visual representation. It must always remain a valid position within the current document. Its column is a logical character position within the line — one per Unicode code point, including TAB — not a terminal screen column or a UTF-8 byte offset; visual expansion of characters such as TAB belongs to the Renderer.

**First Introduced**

Volume III, Chapter 14

**Related Terms**

- Preferred Column
- Viewport
- Renderer
- Text Buffer

**Notes**

Architectural overview: Volume II, Chapter 8. Implementation: Volume III, Chapter 14.

# D

## Data Flow

**Definition**

The predictable path that information follows through a software system as it responds to user actions. In the editor, data flow describes how a key press moves from the keyboard through the Input Handler, Input Parser, Input Engine, Command Engine, and Editor State, and finally to the Renderer and Terminal Display.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Architecture (Software Architecture)
- Editor State
  - Event Loop

## Derived State

**Definition**

State that is computed or produced from another source rather than maintained independently. Derived state can always be rebuilt from the state that produces it, so it does not need to be kept separately in sync. In the editor, the Screen Buffer is derived state: it is produced from the current Editor State each rendering cycle and is not an authoritative model of the document or cursor.

**First Introduced**

Volume IV, Chapter 18

**Related Terms**

- Editor State
- Single Source of Truth
- State Ownership
- Editor Screen Buffer

## Dirty Flag

**Definition**

A boolean value that records whether a document has been modified since it was last saved. The dirty flag is typically false when the document matches the file on disk and true when unsaved changes exist, allowing the editor to warn the user before exiting and to avoid unnecessary save operations.

**First Introduced**

Volume III, Chapter 12

**Related Terms**

- Document
- Editor State
- File Manager

## Dirty Region

**Definition**

A portion of the screen whose current visual representation no longer matches the desired state and therefore must be regenerated. Dirty-region rendering (also called dirty tracking) records which cells or regions have changed so that only those parts of the frame are updated. This rendering concept is distinct from the document's Dirty Flag, which records unsaved changes to the file.

**First Introduced**

Volume IV, Chapter 18

**Related Terms**

- Dirty Flag
- Incremental Rendering
- Full Redraw
- Editor Screen Buffer

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

## Document Area

**Definition**

The region of the screen that displays the visible portion of the Document. The document area occupies the terminal rows above the fixed interface elements (the Status Bar and Message Bar), and its height is the terminal height minus those reserved rows. The Renderer draws the visible document lines into this region, stopping at its lower boundary.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Renderer
- Viewport
- Document
- Status Bar
- Message Bar

## Document Coordinates

**Definition**

The logical coordinate system that identifies positions within the Document using a row and a column. Document coordinates describe where a character or the Cursor is within the text, independent of which portion of the document is currently visible on screen. The Renderer translates document coordinates into screen coordinates when building a frame.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Screen Coordinates
- Viewport
- Cursor
- Renderer

## Durability

**Definition**

The property that a successfully completed change remains preserved after the system subsequently experiences a failure such as a crash or power loss. Atomicity describes how a change happens; durability describes whether the successful change survives a failure. A successful atomic replacement alone does not guarantee durability—stronger guarantees may require synchronization mechanisms such as `fsync()`.

**First Introduced**

Volume V, Chapter 23

**Related Terms**

- Atomicity
- Atomic Replacement

## Double Buffering

**Definition**

A technique for avoiding partially constructed frames by separating the frame being constructed from the frame being displayed. The application builds the next frame in a back buffer while the front buffer remains displayed, then presents the completed frame as a single result. In a terminal editor, this principle is applied at the application level using the Editor Screen Buffer rather than a hardware framebuffer.

**First Introduced**

Volume IV, Chapter 17

**Related Terms**

- Editor Screen Buffer
- Immediate-Mode Rendering
- Full Redraw
- Frame

## Echo

**Definition**

A terminal behavior, enabled by default in canonical mode, where each character typed by the user is immediately displayed on the screen by the terminal driver. When echo is disabled in raw mode, the application becomes responsible for determining what, if anything, appears on screen in response to keyboard input.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Canonical Mode
- Raw Mode
- termios

## Dynamic Array

**Definition**

An array whose capacity can change at runtime. It maintains a reference to its contiguous storage together with an explicit length and capacity; when an insertion requires more space than the current capacity, it grows by allocating a larger region, copying the existing elements into it, and releasing the old storage. Growing by a substantial factor (such as doubling) keeps resizes rare, so the amortized cost of appending stays low. Resizing does not change the fundamentals of the representation: the elements remain contiguous within each allocation.

**First Introduced**

Volume VI, Chapter 25

**Related Terms**

- Array
- Fixed-Size Array
- Capacity
- Length
- Resizing
- Amortized Complexity
- Vector of Lines

## Dynamic String

**Definition**

A growable sequence of characters built from the dynamic-array model. It stores three pieces of information: `data`, a pointer to contiguous character storage; `length`, the number of characters currently in the string; and `capacity`, the amount of storage available. Unlike a C string, it knows its length explicitly rather than scanning for a null terminator, and its allocation can grow as characters are added. The same pattern of pointer, logical size, and allocated capacity describes many growable collections in systems programming.

**First Introduced**

Volume VI, Chapter 25

**Related Terms**

- C String
- Null Terminator
- Dynamic Array
- Length
- Capacity
- Array
- Vector of Lines

# E

## Editing Locality

**Definition**

The property of interactive text editing that users tend to perform many consecutive edits near the current cursor position before moving elsewhere. The gap buffer is designed around editing locality: while editing remains local, the gap stays positioned at the cursor and operations stay cheap, but large cursor movements force the gap to relocate, paying a cost proportional to the distance traveled. Editing locality is temporal locality applied to document positions rather than memory addresses.

**First Introduced**

Volume VI, Chapter 26

**Related Terms**

- Temporal Locality
- Spatial Locality
- Gap Buffer
- Cursor

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

## Editor Screen Buffer

**Definition**

The editor's own application-side, in-memory representation of the frame currently being constructed, before that frame is converted into terminal output. It is part of the editor's rendering pipeline and does not become a second authoritative model of the document or editor state.

**First Introduced**

Volume IV, Chapter 17

**Related Terms**

- Screen Buffer
- Alternate Screen Buffer
- Renderer
- Frame
- Screen Cell

**Notes**

Distinct from the terminal emulator's screen buffers (the normal and alternate screen buffers), which belong to the terminal and are controlled through terminal output. The data structure itself is examined in Chapter 18.

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

## errno

**Definition**

A variable the operating system sets when many system calls fail, identifying the reason for the failure. The File Manager inspects `errno` after a failed operation and converts it into a human-readable message using functions such as `strerror()` or `perror()`, rather than exposing the raw numeric value to the user.

**First Introduced**

Volume V, Chapter 21

**Related Terms**

- File Manager
- File Descriptor
- File Permission

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

The component responsible for loading documents from disk into memory and saving the current document back to disk. The File Manager reads and writes files through operating system interfaces and reports the result of file operations to the rest of the editor, which updates the Editor State accordingly — for example, the Command Engine clears the Dirty Flag only after the File Manager reports a successful save.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Document
- Editor State
- Command Engine
- Dirty Flag

**Notes**

Architectural overview: Volume II, Chapter 8. Detailed treatment: Volume V, Chapter 21.

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

**Notes**

Detailed treatment: Volume V, Chapter 21.

## File Permission

**Definition**

Rules enforced by the operating system that determine whether a particular user may read, write, or otherwise access a file. Reading and writing are separate permissions, so a file may be loadable but not writable; the operating system — not the editor — remains the authority over these restrictions.

**First Introduced**

Volume V, Chapter 21

**Related Terms**

- File Manager
- File Descriptor
- errno

## Fixed-Size Array

**Definition**

An array whose capacity is determined when its storage is created and cannot change for the lifetime of the array. A fixed-size array is simple to understand, allocate, and index, predictable in memory usage, and efficient when the required size is known in advance. It cannot, however, represent text that grows while the user types, so editable document lines require the variable capacity of a dynamic array; fixed sizes remain appropriate for bounded data such as the editor's screen dimensions.

**First Introduced**

Volume VI, Chapter 25

**Related Terms**

- Array
- Dynamic Array
- Capacity
- Length
- Contiguous Storage

## Frame

**Definition**

A single rendered representation of the editor displayed on the terminal.

Each iteration of the event loop typically produces one frame.

**First Introduced**

Volume IV, Chapter 17

**Related Terms**

- Renderer
- Event Loop
- Immediate-Mode Rendering
- Retained-Mode Rendering

**Notes**

Architectural overview: Volume II, Chapter 8. Implementation: Volume IV, Chapter 17.

## Frame Generation

**Definition**

The process of transforming the current Editor State into a complete visual frame stored in the Screen Buffer. Frame generation reads the editor's logical state and populates the buffer with the visible document content, interface elements, and cursor, without modifying the state it reads. The result is a complete snapshot of the screen ready to be converted into terminal output.

**First Introduced**

Volume IV, Chapter 18

**Related Terms**

- Frame
- Editor Screen Buffer
- Editor State
- Renderer

## Full Redraw

**Definition**

The rendering strategy of regenerating the entire visible interface from the current editor state on every frame, rather than incrementally updating only the regions of the screen that changed. This avoids the complexity of tracking differences between frames.

**First Introduced**

Volume I, Chapter 4

**Related Terms**

- Rendering Cycle
- Frame
- Renderer
- Immediate-Mode Rendering

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

## Gap

**Definition**

The unused region reserved inside a gap buffer's underlying array at the current editing position. The gap is storage, not document text: it is invisible to the logical document, and characters before and after it form the logical text. Insertion consumes the gap, deletion enlarges it, and moving the gap relocates it without changing the document. The gap exists to absorb repeated local edits so that the characters that follow the cursor do not have to shift.

**First Introduced**

Volume VI, Chapter 26

**Related Terms**

- Gap Buffer
- Gap Buffer Invariant
- Logical Position
- Physical Position

**Notes**

Conceptual overview: Volume III, Chapter 13. Detailed treatment: Volume VI, Chapter 26.

## Gap Buffer

**Definition**

A text representation that stores the document as a contiguous character sequence while reserving an empty region—the gap—at the current editing position. Characters are inserted into the gap, and the gap is moved when the cursor moves, so repeated edits near the same position avoid shifting the entire document. The gap must be moved and resized as needed, adding implementation complexity.

**First Introduced**

Volume III, Chapter 13

**Related Terms**

- Gap
- Gap Buffer Invariant
- Logical Position
- Physical Position
- Vector of Lines
- Piece Table
- Rope
- Text Buffer

**Notes**

Conceptual overview: Volume III, Chapter 13. Detailed implementation: Volume VI, Chapter 26.

## Gap Buffer Invariant

**Definition**

The rule that a valid gap buffer must always satisfy: `0 ≤ gap_start ≤ gap_end ≤ capacity`. Every position before `gap_start` contains document data, every position from `gap_start` up to `gap_end` is unused, and every position from `gap_end` to `capacity` contains document data. Insertion, deletion, gap movement, and resizing must each leave the buffer satisfying this invariant.

**First Introduced**

Volume VI, Chapter 26

**Related Terms**

- Gap
- Gap Buffer
- Capacity
- Logical Position
- Physical Position
- Contiguous Storage

## Game Loop

**Definition**

A repetitive loop used by interactive applications that continuously read input, update internal state, and render the result. Games run this loop continuously because their world may change even without input; a text editor is event-driven instead, waiting for an event before updating state and rendering a single frame.

**First Introduced**

Volume IV, Chapter 17

**Related Terms**

- Event Loop
- Event-Driven Architecture
- Rendering Cycle
- Frame

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

## Horizontal Scrolling

**Definition**

Scrolling that changes the Viewport's `viewport_column` offset to control which visual/display columns of a long line are visible. Because the editor does not use soft wrapping, a long line remains a single document line and horizontal scrolling exposes different portions of it as the Cursor moves horizontally.

**First Introduced**

Volume IV, Chapter 20

**Related Terms**

- Scrolling
- Vertical Scrolling
- Viewport
- Soft Wrapping

## Horizontal Tab

**Definition**

The Unicode control character **U+0009** (also written `\t`), which the editor stores in the Document as a single logical character, exactly like `A` or `7`. In UTF-8 a Horizontal Tab is encoded as the single byte `09`. It is one code point, so it occupies one logical column; how many terminal cells it spans when displayed is decided by the Renderer's tab expansion, never by the Document or the encoding.

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- Control Character
- Code Point
- UTF-8
- ASCII

# I

## Immediate-Mode Rendering

**Definition**

A rendering model in which the application derives the visual output directly from the current application state each time it renders, keeping no persistent representation of the interface. Given the same relevant state, the renderer produces the same result. Immediate mode is simpler to reason about than retained-mode rendering but may repeat rendering work for unchanged regions.

**First Introduced**

Volume IV, Chapter 17

**Related Terms**

- Retained-Mode Rendering
- Full Redraw
- Frame
- Renderer

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
- Retained-Mode Rendering

## Input Engine

**Definition**

The component responsible for translating abstract key events into editor commands and dispatching those commands to the Command Engine.

It does not read raw terminal bytes, parse escape sequences, or modify the Document. It determines which command an input event represents and passes that command along for execution.

**First Introduced**

Volume III, Chapter 15

**Related Terms**

- Input Handler
- Input Parser
- Command Engine
- Editor State

**Notes**

The Input Engine is the third of three input stages. The Input Handler reads raw bytes, the Input Parser translates those bytes into key events, and the Input Engine maps those events to editor commands. Command execution itself belongs to the Command Engine.

## Input Handler

**Definition**

The component responsible for reading raw keyboard input from the terminal.

It does not interpret escape sequences, map events to commands, or directly modify the document.

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Input Parser
- Input Engine
- Command Engine
- Event Loop

**Notes**

Architectural overview: Volume II, Chapter 8.

The Input Handler is the first of three input stages. It reads raw bytes; the Input Parser (introduced in Volume I, Chapter 6) translates those bytes into key events; the Input Engine (introduced in Volume III, Chapter 15) maps those events to editor commands.

## Input Parser

**Definition**

The component responsible for translating the raw byte stream received by the Input Handler into abstract key events. The parser examines incoming bytes, recognizes patterns such as escape sequences and multi-byte UTF-8 characters, and produces structured events (e.g., `KEY_ARROW_UP`, `KEY_CTRL_S`) that the rest of the editor can consume without knowledge of terminal protocols.

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- Input Handler
- Input Engine
- Input Parsing
- Escape Character (ESC)
- Lookahead

**Notes**

The Input Parser is the middle of three input stages. The Input Handler (introduced in Volume II, Chapter 8) reads raw bytes; the Input Parser translates those bytes into key events; the Input Engine (introduced in Volume III, Chapter 15) maps those events to editor commands.

## Input Size

**Definition**

The measure of the input that is relevant to a particular operation. An editor has several such measures—the number of document lines, the length of an affected line, the total number of characters, the encoded file size in bytes, or the visible screen dimensions—so the input size must be defined before an operation's complexity can be described.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Complexity Analysis
- Time Complexity

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

## Internal Node

**Definition**

In a Rope, a node that organizes other nodes rather than containing document text itself. An internal node connects its children and may hold metadata such as a subtree weight describing how much text lies below it. The root is typically an internal node, while the leaves it organizes hold the actual text fragments.

**First Introduced**

Volume VI, Chapter 28

**Related Terms**

- Rope
- Leaf Node
- Subtree Weight
- Rope Balancing
- Logical Document

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

## Key Binding

**Definition**

The association between an input event and the editor command it triggers. A key binding records which command a given key combination represents; it is the responsibility of the Input Engine to recognize the event and map it to the command, while execution belongs to the Command Engine.

**First Introduced**

Volume III, Chapter 15

**Related Terms**

- Keyboard Shortcut
- Key Event
- Input Engine
- Command

## Key Event

**Definition**

A structured, editor-level representation of a single piece of user input, produced by the Input Parser from raw terminal bytes. A key event describes what was entered (for example, `KEY_ARROW_LEFT`, `KEY_BACKSPACE`, or `KEY_CHARACTER('a')`) rather than how the terminal encoded it, so higher layers can process input without knowledge of terminal protocols.

**First Introduced**

Volume I, Chapter 6

**Related Terms**

- Input Parser
- Input Engine
- Event
- Command

**Notes**

Key events are also called input events. They form a contract between the Input Parser, which produces them, and the Input Engine, which maps them to commands.

## Keyboard Shortcut

**Definition**

A key combination (such as Ctrl+S or Ctrl+Q) that has been bound to an editor command. A keyboard shortcut is a binding between an input event and a command, not the operation itself: changing the binding changes how the command is triggered without changing what the command does.

**First Introduced**

Volume III, Chapter 15

**Related Terms**

- Key Binding
- Key Event
- Command
- Input Engine

# L

## Latency

**Definition**

The delay between an action and the corresponding result becoming visible to the user. Complexity describes how work scales, while latency is the observed delay for a particular execution, so an expensive operation inside one event-loop iteration becomes visible as latency between a key press and the updated screen.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Time Complexity
- Complexity Analysis
- Event Loop

## Layered Architecture

**Definition**

An architectural pattern that organizes software components into distinct levels of responsibility, where each layer has a specific purpose and dependencies flow in a well-defined direction. In the editor, the layers are the Application Layer (command processing), the Core Layer (editor state and data), and the Platform Layer (terminal, file, and input services).

**First Introduced**

Volume II, Chapter 8

**Related Terms**

- Architecture (Software Architecture)
- Low Coupling
- Module

## Leaf Granularity

**Definition**

The chosen size of the text fragments stored in a Rope's leaves. Very small leaves reduce the amount of text handled in a local edit but create many nodes and structural overhead, while very large leaves reduce tree overhead but make modifying text inside a leaf more expensive. The appropriate granularity balances these costs and depends on the workload rather than on a single universally correct value.

**First Introduced**

Volume VI, Chapter 28

**Related Terms**

- Rope
- Leaf Node
- Internal Node
- Rope Balancing
- Cache Locality

## Leaf Node

**Definition**

In a Rope, a node at the bottom of the tree that stores an actual fragment of document text. Leaves are the nodes that physically hold the document's characters, so reading the leaves from left to right reproduces the logical document. This contrasts with internal nodes, which organize fragments but do not themselves contain document text.

**First Introduced**

Volume VI, Chapter 28

**Related Terms**

- Rope
- Internal Node
- Leaf Granularity
- Subtree Weight
- Logical Document

## Length

**Definition**

The number of elements currently belonging to a sequence, as opposed to the capacity, which is the amount of storage available. In a dynamic array or dynamic string with unused capacity, the length is smaller than the capacity, and operations such as appending change the length without changing the allocation. A C string, by contrast, does not store its length and must find it by scanning for the null terminator.

**First Introduced**

Volume VI, Chapter 25

**Related Terms**

- Capacity
- Fixed-Size Array
- Dynamic Array
- Dynamic String
- C String
- Null Terminator

## Linker

**Definition**

The tool that combines multiple object files and libraries into a single executable program. The linker resolves symbol references (function calls and variable accesses) between separately compiled modules by matching each reference to its definition, producing a complete program that the operating system can load and execute.

**First Introduced**

Volume II, Chapter 10

**Related Terms**

- Object File
- Build System
- Compilation

## Linear Time

**Definition**

The complexity classification of an operation whose work grows proportionally to its input size, written `O(n)`. Many text-processing operations that scan the data they touch have linear time. The notation describes the growth rate, not an exact instruction count.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Big-O Notation
- Constant Time
- Quadratic Time
- Time Complexity

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

## Line Ending

**Definition**

The byte sequence that marks where one logical line ends and the next begins in a file. The two conventions our editor recognizes are **LF** — the single byte `0A` (Line Feed), used on Unix/Linux — and **CRLF** — the two bytes `0D 0A` (Carriage Return followed by Line Feed), used on Windows. A line ending belongs to the file representation, not to the Document's logical lines: the editor stores lines without any line-ending bytes, recognizes both LF and CRLF when loading, and writes a single canonical LF ending when saving.

**First Introduced**

Volume V, Chapter 22

**Related Terms**

- Loading
- Saving
- Serialization
- File Manager

**Notes**

The canonical-LF save policy is recorded in ADR-021: Text Encoding Strategy.

## Loading

**Definition**

The one-way transfer of a file's contents from persistent storage into the in-memory Document. Loading opens the file, reads its UTF-8 bytes, decodes those bytes into Unicode code points to construct the Document, and closes the file descriptor; the file on disk remains unchanged until the user explicitly saves.

**First Introduced**

Volume V, Chapter 21

**Related Terms**

- Saving
- File Manager
- Document
- File Descriptor

## Logical Column

**Definition**

A horizontal position measured in Unicode code points within a logical Document line. The Cursor's column is a logical column: one per code point, including TAB (U+0009), and never a UTF-8 byte offset or a terminal screen column. A logical column becomes a visual column only when the Renderer prepares a frame.

**First Introduced**

Volume III, Chapter 14

**Related Terms**

- Visual/Display Column
- Screen Column
- Code Point
- Cursor
- Logical Position

## Logical Document

**Definition**

The current document text as the user and editor understand it: a single continuous sequence of characters that is independent of how the text is physically represented. In a Piece Table, the logical document is assembled by traversing the ordered piece sequence and reading the referenced ranges from the Original Buffer and Add Buffer, so it does not need to occupy one contiguous region of memory. In a Rope, the same logical sequence is reproduced by reading the leaves from left to right across the tree. In every representation the logical document can change through editing, and because the editor may store it in pieces rather than as contiguous characters, its physical layout can diverge from its logical sequence.

**First Introduced**

Volume VI, Chapter 27

**Related Terms**

- Piece Table
- Piece Descriptor
- Piece Sequence
- Original Buffer
- Add Buffer
- Rope
- Leaf Node
- Logical Position

## Logical Position

**Definition**

A position within the document as the user understands it, where characters form one continuous sequence. A logical position is independent of how the text is physically stored: in a gap buffer it differs from the physical position whenever the gap has any size, because the unused region splits the document's characters across both sides of the array, and in a piece table it corresponds to a place inside the ordered piece sequence rather than a buffer offset. Renderers and document operations must treat the logical sequence as the document while the representation manages the physical layout.

**First Introduced**

Volume VI, Chapter 26

**Related Terms**

- Physical Position
- Piece Sequence
- Piece Table
- Gap Buffer
- Gap
- Logical Column

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

## Memline

**Definition**

Vim's internal system for storing buffer text. Vim's conceptual model of a buffer is line-oriented, but Memline runs underneath it, organizing text using line segments and supporting efficient access to the lines that make up a buffer. Memline therefore demonstrates that an editor's simple conceptual model does not have to match its underlying storage machinery: developers can think in terms of lines and cursor positions while the implementation uses more sophisticated storage beneath.

**First Introduced**

Volume VI, Chapter 29

**Related Terms**

- Buffer
- Text Buffer
- Vector of Lines
- Text Buffer Workload

## Message Bar

**Definition**

The fixed interface region at the bottom of the screen that displays short-lived status and error messages, such as "File saved" or "Unable to open file". The Message Bar presents the current status message from the Editor State and is cleared when no message exists; it never becomes part of the Document and never affects the Dirty Flag.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Status Bar
- Renderer
- Editor State
- Dirty Flag

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

## Null Terminator

**Definition**

The zero-valued byte `'\0'` that marks the end of a C string in memory. Because a C string does not store its length, C functions determine where the string ends by scanning until they encounter the terminator. The terminator is part of the storage representation — it consumes a byte of the allocation — rather than part of the logical text, and it is distinct from the character `'0'`. A dynamic string that tracks its length explicitly does not need to scan for a terminator.

**First Introduced**

Volume VI, Chapter 25

**Related Terms**

- C String
- Length
- Dynamic String
- UTF-8

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

## Original Buffer

**Definition**

The buffer in a Piece Table that holds the document's initial contents, exactly as they were when the file was loaded. During normal editing the Original Buffer is never modified: pieces reference ranges of it, and new text is placed in the Add Buffer instead. This immutability makes the Original Buffer a stable source of text whose positions can be referenced by pieces for the lifetime of the document.

**First Introduced**

Volume VI, Chapter 27

**Related Terms**

- Piece Table
- Add Buffer
- Piece Descriptor
- Logical Document

# P

## Off-Screen Rendering

**Definition**

The technique of constructing a complete visual frame in application-managed memory before sending it to the actual display. In the editor, the Screen Buffer serves as the off-screen surface: the renderer draws the complete frame into it, and only then is the result converted into terminal output. This separates frame construction from presentation and prevents the terminal from observing a partially built frame.

**First Introduced**

Volume IV, Chapter 18

**Related Terms**

- Editor Screen Buffer
- Double Buffering
- Full Redraw
- Frame

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

## Physical Position

**Definition**

An index within the underlying array where a character is actually stored, as opposed to its position in the logical document. In a gap buffer the two coordinate systems differ because the unused gap shifts the physical index of every character after the gap by the size of the gap. A gap buffer must translate between logical and physical positions whenever it reads, writes, or interprets document data.

**First Introduced**

Volume VI, Chapter 26

**Related Terms**

- Logical Position
- Gap Buffer
- Gap
- Contiguous Storage

## Piece Descriptor

**Definition**

A small piece of metadata in a Piece Table that identifies a contiguous range of text inside one of the underlying buffers. A descriptor records the `source` (which buffer contains the text), the `offset` (where the range begins in that buffer), and the `length` (how many characters the range covers). A piece descriptor references text rather than containing it, so editing changes which ranges are included and how they are ordered without rewriting the stored characters.

**First Introduced**

Volume VI, Chapter 27

**Related Terms**

- Piece Table
- Piece Sequence
- Original Buffer
- Add Buffer
- Logical Document

## Piece Fragmentation

**Definition**

The structural growth of a Piece Table in which repeated edits divide larger pieces into many smaller ones, increasing the number of piece descriptors needed to describe the current document. Fragmentation is a growth in the *description* of the document, not in the buffers themselves: the stored text remains contiguous while the piece sequence becomes more fragmented. Because position lookup and traversal operate on pieces, a fragmented Piece Table pays increasing metadata and traversal costs over a long editing session.

**First Introduced**

Volume VI, Chapter 27

**Related Terms**

- Piece Table
- Piece Descriptor
- Piece Sequence
- Logical Document

## Piece Sequence

**Definition**

The ordered list of piece descriptors in a Piece Table that determines the current logical document. Following the pieces in order and reading the referenced ranges of text produces the document; the buffers alone do not define the document's order. Insertions and deletions alter this sequence — adding, removing, splitting, or reordering descriptors — while leaving the underlying buffers stable.

**First Introduced**

Volume VI, Chapter 27

**Related Terms**

- Piece Table
- Piece Descriptor
- Original Buffer
- Add Buffer
- Logical Document

## Piece Table

**Definition**

A text representation that keeps the document's text in one or more underlying buffers and represents the current document as an ordered sequence of pieces, each referring to a portion of a buffer. Editing operations record new pieces rather than rewriting existing text, which can make insertions and deletions efficient while preserving the original document data. The added indirection makes the representation more complex than storing text directly.

**First Introduced**

Volume III, Chapter 13

**Related Terms**

- Original Buffer
- Add Buffer
- Piece Descriptor
- Piece Sequence
- Piece Fragmentation
- Gap Buffer
- Vector of Lines
- Rope
- Text Buffer

**Notes**

Conceptual overview: Volume III, Chapter 13. Detailed implementation: Volume VI, Chapter 27.

## Piece Table Invariant

**Definition**

A rule that must remain true before and after every operation for a Piece Table to remain a valid representation of the document. The core invariants are: every piece references a valid range within its source buffer; the Original Buffer is never modified; the Add Buffer only grows; piece order defines document order; each piece's length determines exactly how much text it represents; and the logical document is the concatenation of all active pieces in sequence. Checking these invariants turns them from documentation into a practical tool for debugging operations such as splitting and deletion.

**First Introduced**

Volume VI, Chapter 27

**Related Terms**

- Piece Table
- Piece Descriptor
- Piece Sequence
- Original Buffer
- Add Buffer
- Logical Document

## Piece Tree

**Definition**

The text buffer used by Visual Studio Code, described by its developers as a multiple-buffer piece table organized with a red-black tree and optimized around the editor's line-based model. It extends the Piece Table by adding a tree structure, so that navigating and modifying a large collection of pieces remains practical even for documents with thousands or millions of lines and frequent edits at arbitrary locations. The Piece Tree illustrates how more demanding workloads can justify a more sophisticated text representation.

**First Introduced**

Volume VI, Chapter 29

**Related Terms**

- Piece Table
- Piece Descriptor
- Piece Sequence
- Rope
- Text Buffer
- Text Buffer Workload

## Preferred Column

**Definition**

A value stored alongside the cursor that records the horizontal position the user intends to maintain while moving vertically. When the cursor moves up or down onto a line too short to hold that column, the cursor's actual column is temporarily clamped to the end of the line while the preferred column is preserved, so the original horizontal position can be restored on a longer line. Horizontal movement (Left/Right) establishes a new preferred column; vertical movement (Up/Down) preserves it. Like the cursor's column, the preferred column is measured in logical character positions, not in terminal cells.

**First Introduced**

Volume III, Chapter 14

**Related Terms**

- Cursor

## Pointer Chasing

**Definition**

The memory-access pattern that occurs when a data structure stores related objects in separate, non-adjacent memory locations, forcing the processor to follow a pointer before it learns where the next object it needs is located. Pointer-based structures such as a Rope pay this cost repeatedly, and when the referenced objects are not already in cache, each access can require waiting on slower memory. This makes cache locality as important as Big-O when estimating practical performance.

**First Introduced**

Volume VI, Chapter 28

**Related Terms**

- Rope
- Cache Locality
- Spatial Locality
- Temporal Locality

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

# Q

## Quadratic Time

**Definition**

The complexity classification of an operation whose work grows with the square of its input size, written `O(n²)`. A common pattern compares many elements with many other elements, so doubling the input can multiply the amount of work by approximately four.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Big-O Notation
- Constant Time
- Linear Time
- Time Complexity

# R

## Raw Mode

**Definition**

A terminal operating mode where input is delivered immediately to the application without line buffering or automatic processing by the terminal.

**First Introduced**

Volume I, Chapter 5

**Related Terms**

- Canonical Mode
- termios

## Register

**Definition**

A tiny storage location built directly into the CPU used to hold temporary data during instruction execution. Registers are extremely fast but limited in number and capacity.

**First Introduced**

Volume I, Chapter 2

**Related Terms**

- CPU
- Main Memory
- Instruction Cycle

## Renderer

**Definition**

The presentation component of the editor: a read-only projection that transforms the current Editor State into a complete visual frame. The Renderer reads the Document, Cursor, Viewport, and related state and writes the resulting frame into the Screen Buffer, without modifying the state it renders. It does not read input, execute commands, or convert the frame into terminal output — that final step belongs to the terminal-facing layer.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Frame
- Viewport
- Editor Screen Buffer
- Editor State
- Screen Buffer

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

## Rendering Model

**Definition**

The general approach an application uses to construct and update its visual output, describing how the display is derived from application state and when rendering work occurs. The two fundamental models are immediate-mode rendering, which derives output from state on each frame, and retained-mode rendering, which maintains a persistent visual representation updated incrementally.

**First Introduced**

Volume IV, Chapter 17

**Related Terms**

- Immediate-Mode Rendering
- Retained-Mode Rendering
- Frame
- Renderer

## Requirements Traceability

**Definition**

A professional practice in which every architectural component, test case, and implementation feature can be traced back to one or more requirement identifiers. Traceability ensures that every part of the system exists for a defined purpose and helps prevent unnecessary complexity, making it easier to manage changes as a project evolves.

**First Introduced**

Volume II, Chapter 7

**Related Terms**

- Functional Requirement
- Software Requirements Specification (SRS)
- Non-Functional Requirement (NFR)

## Retained-Mode Rendering

**Definition**

A rendering model in which the application maintains a persistent representation of the visual interface and updates it incrementally as state changes, rather than reconstructing the output from scratch each frame. Retained mode can avoid repeating work on unchanged regions but introduces additional state that must remain synchronized with application state.

**First Introduced**

Volume IV, Chapter 17

**Related Terms**

- Immediate-Mode Rendering
- Incremental Rendering
- Frame
- Renderer

## Rope

**Definition**

A text representation that organizes a document as a tree of text fragments rather than as one large contiguous sequence. Each leaf holds a portion of the document, and the tree structure allows the editor to locate and modify regions without moving the entire document, which scales well to very large files. The editor must maintain and balance the tree, making ropes more complex to implement than a vector of lines.

**First Introduced**

Volume III, Chapter 13

**Related Terms**

- Gap Buffer
- Piece Table
- Vector of Lines
- Text Buffer
- Leaf Node
- Internal Node
- Subtree Weight
- Rope Balancing
- Leaf Granularity

**Notes**

Conceptual overview: Volume III, Chapter 13. Detailed implementation: Volume VI, Chapter 28.

## Rope Balancing

**Definition**

The process of maintaining a Rope's tree so that its height stays within a useful bound relative to the amount of text it represents. Repeated insertions, deletions, and concatenations can skew the tree into a chain that degrades navigation and editing from logarithmic toward linear cost. Balancing—achieved through weights, rotations, or occasional rebuilding, rather than perfect symmetry—preserves the performance properties that make a Rope attractive.

**First Introduced**

Volume VI, Chapter 28

**Related Terms**

- Rope
- Internal Node
- Subtree Weight
- Leaf Granularity
- Amortized Complexity

# S

## Space Complexity

**Definition**

A measure of how the memory required by an operation grows with its input. The editor distinguishes the memory occupied by the existing document from the additional memory an operation requires—such as the `O(1)` additional space of a few temporary variables or the `O(C)` space of a complete temporary copy. Reducing one resource cost can increase another, so time and space must be considered together.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Time Complexity
- Big-O Notation
- Complexity Analysis
- Cache Locality

## Spatial Locality

**Definition**

The property that when a program accesses one memory location, it is likely to access nearby memory locations soon afterward. Sequential processing of a contiguous structure such as an array exhibits strong spatial locality and makes effective use of the CPU cache.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Temporal Locality
- Cache Locality

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
- Editor Screen Buffer
- Terminal Emulator
- ANSI Escape Sequence

## Screen Cell

**Definition**

A single position in the Screen Buffer, corresponding to one visible character position on the terminal. A screen cell can hold the character to display along with optional visual attributes such as foreground and background. Screen cells are addressed by screen coordinates (row, column), which describe positions on the screen rather than positions in the document.

**First Introduced**

Volume IV, Chapter 18

**Related Terms**

- Editor Screen Buffer
- Character Cell
- Screen Buffer
- Frame

**Notes**

Distinct from the terminal emulator's Character Cell, which belongs to the terminal. The Screen Buffer is composed of screen cells and is examined in detail in Chapter 18.

## Screen Column

**Definition**

A horizontal position within the current terminal frame, measured in terminal cells. Screen columns describe where content appears on screen, and the same document position can map to a different screen column as the Viewport changes. The Renderer produces a screen column by first expanding a logical column into a visual column and then subtracting `viewport_column`.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Screen Coordinates
- Visual/Display Column
- Logical Column
- Viewport

## Screen Coordinates

**Definition**

The coordinate system that identifies positions within the terminal display, using a screen row and a screen column. Screen coordinates describe where content appears in the current frame; the same document position can appear at different screen coordinates as the Viewport changes. The Renderer translates document coordinates into screen coordinates when building a frame.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Document Coordinates
- Viewport
- Screen Cell
- Renderer

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

## Scrolling

**Definition**

The process of changing the Viewport's offsets so that the Cursor remains within the visible region of the terminal. When the Cursor's logical position moves outside the visible area, scrolling adjusts `viewport_row` and `viewport_column`; scrolling is not cursor movement, and the Document itself never changes.

**First Introduced**

Volume IV, Chapter 20

**Related Terms**

- Viewport
- Vertical Scrolling
- Horizontal Scrolling
- Cursor
- Renderer

## Saving

**Definition**

The explicit operation that serializes the Document and writes its contents to persistent storage. Saving is distinct from editing: editing changes the in-memory Document, while saving makes those changes persistent. The Dirty Flag is cleared only after a successful save.

**First Introduced**

Volume V, Chapter 21

**Related Terms**

- Loading
- Serialization
- File Manager
- Dirty Flag

## Serialization

**Definition**

The process of encoding the Document's Unicode code points into the UTF-8 byte sequence that will be written to the file. Serialization is the step that turns the editor's structured Document — a collection of lines of code points — into the flat sequence of bytes a file stores.

**First Introduced**

Volume V, Chapter 21

**Related Terms**

- Saving
- Document
- Loading

## Shell

**Definition**

A command-line interpreter that reads user commands and launches programs. The shell itself does not render output—it starts processes and communicates with them through standard streams, while the terminal emulator handles display.

**First Introduced**

Volume I, Chapter 3

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

## Soft Wrapping

**Definition**

A display strategy in which a long document line is shown across multiple terminal rows while the underlying Document still contains a single line. Wrapping exists only for presentation, so it complicates the mapping between document coordinates and screen coordinates. Our editor uses horizontal scrolling instead, which preserves the simple row-and-column model.

**First Introduced**

Volume IV, Chapter 20

**Related Terms**

- Horizontal Scrolling
- Viewport
- Document Coordinates
- Screen Coordinates

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

An architectural principle where each piece of state has exactly one owner responsible for maintaining and modifying it. In the editor, each field of the Editor State is written by a single component: the Command Engine owns the Document, Viewport, Dirty Flag, and Configuration; the Cursor owns the cursor position; the File Manager owns the current file; the Renderer owns the status message; and the Terminal Interface owns the screen size. Other components may read a field but must not modify it directly.

Ownership answers _who may write a field_ and is distinct from domain responsibility — _who defines the rules and representation of a value_. The Command Engine writes the Document field, while the Document Model owns the representation and editing rules for that text.

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

Volume III, Chapter 12

**Related Terms**

- Application State
- Stateless Program
- Editor State

## Stateless Program

**Definition**

A program that performs a task using the information it receives, produces a result, and terminates without needing to remember anything afterward. Each execution is independent of previous runs and starts from the same initial state. Command-line utilities that take input, produce output, and exit are typically stateless.

**First Introduced**

Volume III, Chapter 12

**Related Terms**

- Application State
- Stateful Program

## Status Bar

**Definition**

The fixed interface region below the document area that displays persistent information about the current editing session, such as the current filename, the modified state (Dirty Flag), and the cursor position. The Status Bar is derived entirely from Editor State and is rendered without modifying that state.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Message Bar
- Dirty Flag
- Cursor
- Renderer
- Editor State

**Notes**

Detailed treatment: Volume IV, Chapter 21.

## Subtree Weight

**Definition**

Metadata attached to a node in a Rope that records how many logical characters are represented by that node's subtree. Because each internal node knows how much text lies in its children, a Rope can locate a logical document position by comparing it against those weights and descending the appropriate branch, skipping over whole regions without reading every preceding character. A subtree weight is navigation information, not document text.

**First Introduced**

Volume VI, Chapter 28

**Related Terms**

- Rope
- Internal Node
- Leaf Node
- Logical Position
- Rope Balancing

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

## Temporal Locality

**Definition**

The property that data accessed recently is likely to be accessed again soon. Repeatedly reusing the same data, such as a frequently read value, exhibits temporal locality and allows the CPU cache to serve the accesses quickly.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Spatial Locality
- Cache Locality
- Editing Locality

## Temporary File

**Definition**

An intermediate file created during a save operation to construct the complete new version of the document without modifying the existing destination. The editor writes the new contents to the Temporary File and only replaces the destination with it after the write succeeds. The Temporary File is not a second editable Document; it is a persistence mechanism used during the save operation.

**First Introduced**

Volume V, Chapter 23

**Related Terms**

- File Manager
- Atomic Replacement
- Document

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

The in-memory data structure that stores the editable contents of the current document as a collection of lines, each holding its text as a sequence of Unicode code points.

All editing operations modify the Text Buffer rather than the file on disk.

**First Introduced**

Volume III, Chapter 13

**Related Terms**

- Document
- Cursor
- Vector of Lines
- Gap Buffer
- Piece Table
- Rope
- Text Buffer Workload

## Text Buffer Workload

**Definition**

The pattern of document sizes, edit frequencies, edit locations, and supporting operations that a particular editor must handle. Because different editors face different workloads, there is no single "best" text buffer: each text representation is chosen because its trade-offs fit the editor's expected requirements. A small editor working with a single document may favor a simple vector of lines, while an editor handling millions of lines, frequent edits, and many position queries may justify a more sophisticated structure such as a piece tree or a rope.

**First Introduced**

Volume VI, Chapter 29

**Related Terms**

- Text Buffer
- Vector of Lines
- Gap Buffer
- Piece Table
- Piece Tree
- Rope
- Editing Locality

## Time Complexity

**Definition**

A measure of how the amount of computational work performed by an operation grows as its input grows, independent of the specific machine, compiler, or implementation details. Time and space complexity are analyzed separately because additional memory can sometimes be traded for less work.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Space Complexity
- Complexity Analysis
- Big-O Notation
- Latency

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

## Vector of Lines

**Definition**

The document representation adopted by our editor, in which a document is stored as a collection of independently stored text lines, each holding its own sequence of characters (Unicode code points). This model matches the user's line-oriented mental model of text and makes operations such as splitting, merging, rendering, and cursor positioning natural, while keeping implementation simple. Character edits still shift characters within a single line, so very long lines can be costly to edit.

**First Introduced**

Volume III, Chapter 13

**Related Terms**

- Document
- Text Buffer
- Gap Buffer
- Piece Table
- Rope

## Vertical Scrolling

**Definition**

Scrolling that changes the Viewport's `viewport_row` offset to control which rows of the Document are visible in the terminal. When the Cursor moves above or below the visible region, vertical scrolling adjusts `viewport_row` so that the Cursor becomes visible again.

**First Introduced**

Volume IV, Chapter 20

**Related Terms**

- Scrolling
- Horizontal Scrolling
- Viewport
- Cursor

## Viewport

**Definition**

The portion of the Document currently visible in the terminal, identified by two offsets: `viewport_row`, the first visible document row, and `viewport_column`, the first visible visual/display column of the rendered line. `viewport_row` counts document rows; `viewport_column` counts visual columns — the display coordinates the Renderer produces after expanding characters such as TAB — and is not a UTF-8 byte offset or a logical Document character index. The Viewport stores only these offsets rather than a copy of the document, and they change as the user scrolls so that the Cursor remains visible.

**First Introduced**

Volume III, Chapter 12

**Related Terms**

- Renderer
- Cursor
- Scrolling
- Document Coordinates
- Screen Coordinates

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

## Visual/Display Column

**Definition**

The intermediate horizontal coordinate that sits between a logical column and a screen column. A logical column counts code points, while a visual/display column counts terminal cells; the Renderer produces visual columns by expanding characters such as TAB, then subtracts `viewport_column` to reach a screen column. On a line without variable-width characters, every logical column equals its visual column.

**First Introduced**

Volume IV, Chapter 19

**Related Terms**

- Logical Column
- Screen Column
- Renderer
- Horizontal Tab

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

## Worst Case

**Definition**

The possible cost of an operation under the most expensive arrangement of its input or state for an input of a particular size. Worst-case analysis is especially useful for an interactive editor because the maximum work an operation may perform can surface directly as user-visible latency.

**First Introduced**

Volume VI, Chapter 24

**Related Terms**

- Best Case
- Average Case
- Amortized Complexity
- Time Complexity

# Future Terms

The following concepts are expected to be added as the manuscript develops:

- Language Server Protocol (LSP)
- Tree-sitter
- Syntax Highlighting

# Maintenance Rules

1. Every canonical technical term should appear exactly once in this glossary.

2. Definitions should be concise and technology-agnostic where possible.

3. If a definition changes, update the glossary before modifying individual chapters.

4. Do not duplicate glossary definitions inside chapters unless necessary for comprehension.

5. Chapters should use glossary terminology consistently.

6. When introducing a new canonical term, immediately add it to this document.

7. If two terms appear to describe the same concept, prefer the glossary definition and terminology.

End of GLOSSARY.md
