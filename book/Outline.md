# **Volume I — Computing Foundations**

Learn what actually happens before your program even starts.

## **Chapter 1 — Introduction**

- What you'll learn
- Why build a text editor?
- Learning objectives
- Final product
- Project roadmap
- Development methodology
- Recommended environment

## **Chapter 2 — How Computers Display Text**

- CPU
- Memory
- Processes
- stdout
- stdin
- Terminal emulator
- Console vs Terminal
- Character rendering
- UTF-8 basics

## **Chapter 3 — The Unix Terminal**

- History
- Teletype
- TTY
- PTY
- Shell
- Session
- File descriptors
- Process groups

## **Chapter 4 — ANSI Escape Sequences**

- Escape codes
- Cursor movement
- Colors
- Screen clearing
- Alternate screen
- Cursor visibility
- Terminal capabilities

## **Chapter 5 — Raw Terminal Mode**

- Canonical mode
- Raw mode
- termios
- Echo
- Signals
- Control characters
- Restoring terminal state

## **Chapter 6 — Keyboard Input**

- ASCII
- UTF-8 input
- Arrow keys
- Escape sequences
- Function keys
- Ctrl shortcuts
- Parsing input

---

# **Volume II — Software Architecture**

Design before writing code.

## **Chapter 7 — Software Requirements Specification**

- Functional requirements
- Non-functional requirements
- Constraints
- Assumptions
- Success criteria
- Full SRS

## **Chapter 8 — System Architecture**

- Layered architecture
- Module boundaries
- Responsibilities
- Dependencies
- State ownership

## **Chapter 9 — Event Driven Architecture**

- Main loop
- Event processing
- State updates
- Rendering cycle

## **Chapter 10 — Project Structure**

- Directory layout
- Header organization
- Build system
- Module interfaces
- Coding standards

## **Chapter 11 — Design Decisions**

Why:

- Full redraw
- Single document
- No threads
- No plugins
- No syntax highlighting

Tradeoffs.

---

# **Volume III — Core Editor Engine**

## **Chapter 12 — Editor State**

- Designing
- Editor
- Document
- Cursor
- Screen Size
- Status
- Configuration

## **Chapter 13 — Document Model**

- Representing text.
  - Single string
  - Vector
  - Gap Buffer
  - Piece Table
  - Rope

## **Chapter 14 — Cursor Engine**

- Movement
  - Horizontal
  - Vertical
- Preferred column
- Scrolling interaction

## **Chapter 15 — Input Engine**

- Input events
- Key mapping
- Commands
- Shortcut dispatch

## **Chapter 16 — Command Engine**

- Insert
- Delete
- Split line
- Move cursor
- Save
- Quit

---

# **Volume IV — Rendering Engine**

## **Chapter 17 — Rendering Theory**

- Immediate mode
- Retained mode
- Game loops
- Double buffering

## **Chapter 18 — Screen Buffer**

- Off-screen rendering
- Dirty tracking
- Frame generation

## **Chapter 19 — Renderer**

- Drawing lines
- Status bar
- Message bar
- Cursor
- Viewport

## **Chapter 20 — Scrolling**

- Vertical
- Horizontal
- Viewport
- Offsets

---

# **Volume V — File System**

## **Chapter 21 — File I/O**

- Loading
- Saving
- Errors
- Permissions
- Large files

## **Chapter 22 — Text Encoding**

- ASCII
- UTF-8
- Unicode
- Line endings
- CRLF
- LF

## **Chapter 23 — File Safety**

- Temporary files
- Atomic saves
- Recovery
- Crash scenarios

---

# **Volume VI — Algorithms & Data Structures**

This is where the book becomes much deeper than a typical tutorial.

## **Chapter 24 — Complexity Analysis**

- Big-O
- Memory usage
- Cache locality
- Latency

## **Chapter 25 — Arrays and Strings**

- Insertion
- Deletion
- Resizing

## **Chapter 26 — Gap Buffer**

- Theory
- Implementation
- Advantages
- Disadvantages

## **Chapter 27 — Piece Table**

Used by VS Code.

- Architecture
- Operations
- Complexity

## **Chapter 28 — Rope**

Used by very large editors.

- Balancing
- Performance

## **Chapter 29 — Comparing Text Buffers**

Which editor uses what?

- Nano
- Vim
- Emacs
- VS Code
- Helix
- Zed
- Sublime

---

# **Volume VII — Professional Features**

## **Chapter 30 — Search**

- Linear search
- Incremental search
- Regex

## **Chapter 31 — Undo / Redo**

- Command Pattern
- Snapshots
- Operation stacks

## **Chapter 32 — Syntax Highlighting**

- Lexer
- Parser
- Tokens
- Themes

## **Chapter 33 — Configuration**

- Settings
- Key bindings
- Theme

## **Chapter 34 — Multiple Buffers**

- Documents
- Tabs
- Windows

---

# **Volume VIII — Software Engineering**

## **Chapter 35 — Testing**

- Unit tests
- Integration
- Manual
- Regression

## **Chapter 36 — Debugging**

- gdb
- Assertions
- Logging
- Profiling

## **Chapter 37 — Performance**

- Rendering
- Memory
- Large files
- Optimization

## **Chapter 38 — Portability**

- Linux
- macOS
- Windows
- WSL

## **Chapter 39 — Packaging**

- CMake
- Release
- Distribution

---

# **Volume IX — Advanced Topics**

## **Chapter 40 — Plugin Architecture**

- Interfaces
- ABI
- Dynamic loading

## **Chapter 41 — Language Server Protocol (LSP)**

- Overview
- Diagnostics
- Completion
- Hover

## **Chapter 42 — Tree-sitter**

- Incremental parsing
- Syntax trees
- Highlighting

## **Chapter 43 — Terminal Graphics**

- True color
- Images
- Mouse
- Kitty protocol

## **Chapter 44 — Modern Editors**

How these editors are built:

- Nano
- Vim
- Neovim
- Emacs
- Helix
- VS Code
- Sublime Text
- Zed

Architecture comparison.

---

# **Volume X — Building Our Editor**

This is the implementation volume.

## **Chapter 45 — Milestone 1**

Initialize terminal

## **Chapter 46 — Milestone 2**

Renderer

## **Chapter 47 — Milestone 3**

Cursor

## **Chapter 48 — Milestone 4**

Insertion

## **Chapter 49 — Milestone 5**

Deletion

## **Chapter 50 — Milestone 6**

Line splitting

## **Chapter 51 — Milestone 7**

File loading

## **Chapter 52 — Milestone 8**

Saving

## **Chapter 53 — Milestone 9**

Scrolling

## **Chapter 54 — Milestone 10**

Final polish

---

# **Appendices**

### **A — ANSI Escape Reference**

### **B — ASCII Table**

### **C — UTF-8 Reference**

### **D — termios Cheat Sheet**

### **E — POSIX APIs Used**

### **F — Common Terminal Bugs**

### **G — Design Decision Records (ADRs)**

### **H — Glossary**

### **I — Recommended Reading**

### **J — Exercises and Challenge Projects**
