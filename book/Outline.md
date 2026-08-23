# **Volume I — Computing Foundations**

Learn what actually happens before your program even starts.

## **Chapter 1 — Introduction**

- 1.1 Welcome
- 1.2 What You Will Learn
- 1.3 Why Build a Text Editor?
- 1.4 Why a Terminal Editor Instead of a GUI Editor?
- 1.5 Our Philosophy - Learn the "Why"
- 1.6 What We Will Build
- 1.7 What We Will Not Build (Initially)
- 1.8 Who This Book Is For
- 1.9 Recommended Prerequisites
- 1.10 Development Environment
- 1.11 How This Book Is Organized
- 1.12 The Journey Ahead

## **Chapter 2 — How Computers Display Text**

- 2.1 The Journey of a Character
- 2.2 The CPU
- 2.3 Main Memory (RAM)
- 2.4 Processes
- 2.5 Standard Input and Standard Output
- 2.6 From `stdout` to the Terminal
- 2.7 Terminal Emulators
- 2.8 Console vs Terminal
- 2.9 Character Rendering
- 2.10 Character Encoding
- 2.11 Following One Character

## **Chapter 3 — The Unix Terminal**

- 3.1 Why Study the Terminal?
- 3.2 A Brief History of Terminals
- 3.3 What Is a TTY?
- 3.4 Virtual Consoles
- 3.5 Pseudo Terminals (PTYs)
- 3.6 Terminal Emulator
- 3.7 The Shell
- 3.8 Terminal Sessions
- 3.9 Standard File Descriptors
- 3.10 Controlling Terminal
- 3.11 Process Groups
- 3.12 Putting Everything Together

## **Chapter 4 — ANSI Escape Sequences**

- 4.1 The Problem with Plain Text
- 4.2 What Is an ANSI Escape Sequence?
- 4.3 Anatomy of an Escape Sequence
- 4.4 Cursor Movement
- 4.5 Clearing the Screen
- 4.6 Text Formatting
- 4.7 Cursor Visibility
- 4.8 Alternate Screen Buffer
- 4.9 Putting It All Together
- 4.10 Terminal Compatibility
- 4.11 Looking Ahead - Building Our Renderer

## **Chapter 5 — Raw Terminal Mode**

- 5.1 The Problem with Standard Terminal Input
- 5.2 Canonical Mode
- 5.3 Why Editors Need Raw Mode
- 5.4 The `termios` Interface
- 5.5 Line Buffering
- 5.6 Echo
- 5.7 Control Characters and Signals
- 5.8 Raw Mode Flags
- 5.9 Restoring the Terminal
- 5.10 Security and User Experience
- 5.11 Looking Ahead

## **Chapter 6 — Keyboard Input**

- 6.1 From Key Press to Program
- 6.2 Characters vs. Keys
- 6.3 ASCII – The Foundation of Keyboard Input
- 6.4 Control Characters and Ctrl Shortcuts
- 6.5 Beyond ASCII: UTF-8 Input
- 6.6 Escape Sequences for Special Keys
- 6.7 Function Keys and Other Special Keys
- 6.8 Modifier Keys
- 6.9 The Challenge of Parsing Input
- 6.10 Designing an Input Parser

---

# **Volume II — Software Architecture**

Design before writing code.

## **Chapter 7 — Software Requirements Specification**

- 7.1 Why We Need Requirements
- 7.2 What Is a Software Requirements Specification?
- 7.3 Defining the Project Scope
- 7.4 Stakeholders and Users
- 7.5 Functional Requirements
- 7.6 Non-Functional Requirements
- 7.7 Constraints
- 7.8 Assumptions
- 7.9 Use Cases
- 7.10 Success Criteria

## **Chapter 8 — System Architecture**

- 8.1 What Is Software Architecture?
- 8.2 From Requirements to Architecture
- 8.3 Architectural Goals
- 8.4 High-Level Architecture
- 8.5 Layered Architecture
- 8.6 Core Components
- 8.7 Data Flow Through the Editor
- 8.8 State Ownership
- 8.9 Module Dependencies
- 8.10 Architectural Decisions

## **Chapter 9 — Event Driven Architecture**

- 9.1 What Is an Event-Driven System?
- 9.2 The Life of an Editor
- 9.3 Initialization Phase
- 9.4 The Event Loop
- 9.5 Events in Our Editor
- 9.6 Event Processing Pipeline
- 9.7 Commands vs. Events
- 9.8 State Updates
- 9.9 The Rendering Cycle
- 9.10 Maintaining Responsiveness
- 9.11 Error Handling Within the Loop
- 9.12 Putting It All Together

## **Chapter 10 — Project Structure**

- 10.1 Why Project Structure Matters
- 10.2 Mapping Architecture to Source Code
- 10.3 Directory Layout
- 10.4 Module Organization
- 10.5 Header File Organization
- 10.6 Module Interfaces
- 10.7 Shared Types and Common Utilities
- 10.8 Naming Conventions
- 10.9 Build System Overview
- 10.10 Coding Standards

## **Chapter 11 — Design Decisions**

- 11.1 How Engineers Make Design Decisions
- 11.2 Decision 1 - Implement the Editor in C
- 11.3 Decision 2 - Use a Full-Screen Redraw Renderer
- 11.4 Decision 3 - Use Raw Terminal Mode
- 11.5 Decision 4 - Separate Input Parsing from Commands
- 11.6 Decision 5 - Use a Layered Architecture
- 11.7 Decision 6 - Centralize the Editor State
- 11.8 Decision 7 - Give Every Component One Responsibility
- 11.9 Decision 8 - Enforce One-Way Dependencies
- 11.10 Decision 9 - Organize the Project by Modules
- 11.11 Decision 10 - Create a Dedicated Common Module
- 11.12 Decision 11 - Use CMake
- 11.13 Decisions We Intentionally Deferred
- 11.14 Design Principles to Remember

---

# **Volume III — Core Editor Engine**

## **Chapter 12 — Editor State**

- 12.1 What Is Application State?
- 12.2 Stateless vs Stateful Programs
- 12.3 What Information Must Our Editor Remember?
- 12.4 Designing the EditorState Structure
- 12.5 Understanding Each Part
- 12.6 State Ownership
- 12.7 How State Changes During Editing
- 12.8 Designing for Growth

## **Chapter 13 — Document Model**

- 13.1 What Is a Document?
- 13.2 Why Not Edit the File Directly?
- 13.3 Responsibilities of the Document
- 13.4 What Operations Must a Document Support?
- 13.5 Possible Ways to Represent Text
- 13.6 Option 1 – A Single String
- 13.7 Option 2 – A Vector of Lines
- 13.8 Option 3 – Gap Buffer
- 13.9 Option 4 – Piece Table
- 13.10 Option 5 – Rope

## **Chapter 14 — Cursor Engine**

- 14.1 What Is the Cursor?
- 14.2 Cursor Coordinates
- 14.3 Cursor Position vs Character Position
- 14.4 Horizontal Movement
- 14.5 Crossing Line Boundaries
- 14.6 Vertical Movement
- 14.7 Preferred Column
- 14.8 When Preferred Column Changes
- 14.9 Cursor and the Viewport
- 14.10 Keeping the Cursor Visible

## **Chapter 15 — Input Engine**

- 15.1 What the Input Engine Is Responsible For
- 15.2 Input Events as an Abstraction
- 15.3 From Keys to Commands
- 15.4 Key Mapping and Dispatch
- 15.5 Keyboard Shortcuts
- 15.6 Unrecognized Input
- 15.7 Input Engine and Editor State
- 15.8 Input Processing Lifecycle
- 15.9 Design Alternatives
- 15.10 Complexity and Trade-offs

## **Chapter 16 — Command Engine**

- 16.1 What the Command Engine Is Responsible For
- 16.2 Commands as an Abstraction
- 16.3 Command Representation
- 16.4 The Command Execution Lifecycle
- 16.5 Command Categories
- 16.6 Editing Commands and Core Operations
- 16.7 Command Boundaries and State Ownership
- 16.8 Commands That Do Not Modify the Document
- 16.9 Command Results and Error Handling
- 16.10 Command Dispatch
- 16.11 Design Alternatives
- 16.12 Complexity and Trade-offs

---

# **Volume IV — Rendering Engine**

## **Chapter 17 — Rendering Theory**

- 17.1 The Rendering Problem
- 17.2 What Is a Frame?
- 17.3 Immediate-Mode Rendering
- 17.4 Retained-Mode Rendering
- 17.5 Rendering and the Game Loop
- 17.6 Double Buffering
- 17.7 Rendering Models in a Terminal
- 17.8 Design Alternatives
- 17.9 Complexity and Trade-offs

## **Chapter 18 — Screen Buffer**

- 18.1 The Screen Buffer as an Intermediate Representation
- 18.2 What a Screen Cell Represents
- 18.3 Screen Buffer Dimensions
- 18.4 The Screen Buffer and Editor State
- 18.5 Frame Generation
- 18.6 From Frame to Terminal Output
- 18.7 Dirty Tracking and Dirty Regions
- 18.8 Why Full Redraw Makes the Screen Buffer Simple
- 18.9 Screen Buffer Lifecycle
- 18.10 Design Alternatives
- 18.11 Complexity and Trade-offs

## **Chapter 19 — Renderer**

- 19.1 What the Renderer Is Responsible For
- 19.2 From Editor State to a Visual Frame
- 19.3 Rendering the Document Area
- 19.4 Translating Document Coordinates to Screen Coordinates
- 19.5 Rendering the Status Bar
- 19.6 Rendering the Message Bar
- 19.7 Rendering the Cursor
- 19.8 Rendering Order
- 19.9 Design Alternatives
- 19.10 Complexity and Trade-offs

## **Chapter 20 — Scrolling**

- 20.1 Document Coordinates and Screen Coordinates
- 20.2 The Viewport
- 20.3 Vertical Scrolling
- 20.4 Horizontal Scrolling
- 20.5 Keeping the Cursor Visible
- 20.6 Scrolling and Cursor Movement
- 20.7 Scrolling After Editing
- 20.8 Viewport Boundaries
- 20.9 Interaction with the Renderer
- 20.10 Design Alternatives
- 20.11 Complexity and Trade-offs
- 20.12 Common Scrolling Bugs

---

# **Volume V — File System**

## **Chapter 21 — File I/O**

- 21.1 The File Manager's Responsibility
- 21.2 Loading a File
- 21.3 Saving a File
- 21.4 File I/O Errors
- 21.5 File Permissions and the Operating System
- 21.6 Large Files
- 21.7 File I/O in the Runtime Model
- 21.8 Complexity and Trade-offs

## **Chapter 22 — Text Encoding**

- 22.1 Bytes Are Not Characters
- 22.2 Character Sets, Unicode, and Encodings
- 22.3 ASCII
- 22.4 Unicode
- 22.5 UTF-8
- 22.6 UTF-8 and the Text Buffer
- 22.7 Byte Positions vs Character Positions
- 22.8 Valid and Invalid UTF-8
- 22.9 Line Endings
- 22.10 LF vs CRLF
- 22.11 Line Ending Policy
- 22.12 Encoding and Saving
- 22.13 Encoding Strategy for Our Editor
- 22.14 Complexity and Trade-offs

## **Chapter 23 — File Safety**

- 23.1 Why File Safety Matters
- 23.2 What Can Go Wrong During a Save?
- 23.3 Temporary Files
- 23.4 Atomic Replacement
- 23.5 Durability - Atomic Is Not the Same as Durable
- 23.6 Crash Scenarios
- 23.7 Recovery After a Failed Save
- 23.8 What If the Save Fails?
- 23.9 Atomic Replacement and File Metadata
- 23.10 Our File Safety Strategy
- 23.11 What We Are Not Building
- 23.12 Design Alternatives and Trade-offs

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
- Tab width

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