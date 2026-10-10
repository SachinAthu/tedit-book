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
- 23.8 Atomic Replacement and File Metadata
- 23.9 Our File Safety Strategy
- 23.10 What We Are Not Building
- 23.11 Design Alternatives and Trade-offs

---

# **Volume VI — Algorithms & Data Structures**

This is where the book becomes much deeper than a typical tutorial.

## **Chapter 24 — Complexity Analysis**

- 24.1 Why Complexity Matters
- 24.2 What Does "Cost" Mean?
- 24.3 Measuring Input Size
- 24.4 Big-O Notation
- 24.5 Best Case, Average Case, and Worst Case
- 24.6 Time Complexity of Editor Operations
- 24.7 Space Complexity
- 24.8 Cache Locality
- 24.9 Latency in an Interactive Editor
- 24.10 Amortized Complexity

## **Chapter 25 — Arrays and Strings**

- 25.1 Why Arrays Matter to a Text Editor
- 25.2 Contiguous Memory and Indexing
- 25.3 Fixed-Size Arrays
- 25.4 Dynamic Arrays
- 25.5 Growing a Dynamic Array
- 25.6 Array Insertion
- 25.7 Array Deletion
- 25.8 Strings as Arrays of Characters
- 25.9 Building a Dynamic String
- 25.10 Design Alternatives and Trade-offs

## **Chapter 26 — Gap Buffer**

- 26.1 Why a Gap Buffer?
- 26.2 The Basic Idea
- 26.3 Gap Buffer Invariants
- 26.4 Logical Text vs Physical Layout
- 26.5 Moving the Gap
- 26.6 Insertion
- 26.7 Deletion
- 26.8 Growing the Gap
- 26.9 Representing Multiple Lines
- 26.10 Locality of Editing and Cursor Movement
- 26.11 Memory Layout and Cache Behavior
- 26.12 Gap Buffer Failure Modes

## **Chapter 27 — Piece Table**

- 27.1 Why Another Text Buffer?
- 27.2 The Core Idea of a Piece Table
- 27.3 The Two Immutable Buffers
- 27.4 Piece Descriptors
- 27.5 Reconstructing the Document
- 27.6 Inserting Text
- 27.7 Deleting Text
- 27.8 Editing Through Repeated Operations
- 27.9 Piece Table Invariants
- 27.10 Locating a Position in the Piece Table
- 27.11 Complexity and Performance
- 27.12 Strengths, Limitations, and Trade-offs
- 27.13 When Would You Choose a Piece Table?

## **Chapter 28 — Rope**

- 28.1 Why Another Text Representation?
- 28.2 The Central Idea of a Rope
- 28.3 How Text Is Stored
- 28.4 Finding a Position in a Rope
- 28.5 Concatenation and Splitting
- 28.6 Insertion and Deletion
- 28.7 Balancing the Tree
- 28.8 Rope Complexity
- 28.9 Leaf Size and Granularity
- 28.10 Memory Layout and Cache Behavior
- 28.11 Rope Workloads and Scaling

## **Chapter 29 — Comparing Text Buffers**

- 29.1 From Data Structures to Real Editors
- 29.2 Nano
- 29.3 Vim
- 29.4 Emacs
- 29.5 VS Code
- 29.6 Helix and Zed
- 29.7 Sublime Text
- 29.8 Which Editor Uses What?

---

# **Volume VII — Professional Features**

## **Chapter 30 — Search**

- 30.1 Why Search Is More Than Finding Text
- 30.2 The Search Problem
- 30.3 Linear Search
- 30.4 Searching Across Lines
- 30.5 Search State
- 30.6 Incremental Search
- 30.7 Regular Expressions
- 30.8 Regex Changes the Problem
- 30.9 Search User Experience
- 30.10 Search Performance
- 30.11 Design Alternatives and Trade-offs
- 30.12 Common Search Bugs
- 30.13 What We Are Not Building

## **Chapter 31 — Undo / Redo**

- 31.1 Why Undo and Redo Exist
- 31.2 What Does an Undo Operation Mean?
- 31.3 The Snapshot Strategy
- 31.4 The Command Pattern
- 31.5 Operation History
- 31.6 The Two-Stack State Machine
- 31.7 What Counts as an Undoable Action?
- 31.8 Cursor Position and Editor State
- 31.9 Undo, Redo, and the Dirty Flag
- 31.10 Grouping and Coalescing Operations
- 31.11 Memory, Limits, and Trade-offs
- 31.12 Common Undo / Redo Bugs

## **Chapter 32 — Syntax Highlighting**

- 32.1 Why Syntax Highlighting Exists
- 32.2 Syntax Highlighting as an Analysis Pipeline
- 32.3 Lexical Analysis
- 32.4 Tokens and Highlight Classes
- 32.5 Beyond Lexing - Context and Lightweight Parsing
- 32.6 Highlighting the Document and the Text Buffer
- 32.7 Themes and Visual Styles
- 32.8 Language Detection and Highlighting Modes
- 32.9 Updating Highlighting After Editing
- 32.10 Performance and Scalability
- 32.11 Design Alternatives and Trade-offs

## **Chapter 33 — Configuration**

- 33.1 Why Configuration Matters
- 33.2 What Should Be Configurable?
- 33.3 The Configuration Model
- 33.4 Default Configuration
- 33.5 Configuration Sources and Precedence
- 33.6 Configuration File
- 33.7 Configurable Key Bindings
- 33.8 Configuration Validation
- 33.9 Design Alternatives and Trade-offs
- 33.10 Common Configuration Mistakes

## **Chapter 34 — Multiple Buffers**

- 34.1 Why One Document Is No Longer Enough
- 34.2 Document, Buffer, View, Tab, and Window
- 34.3 State Ownership - What Belongs Where
- 34.4 The Buffer Manager and Buffer Lifecycle
- 34.5 Switching Between Buffers
- 34.6 Tabs - Organizing Editing Contexts
- 34.7 Window Layout and Arrangement
- 34.8 Rendering Multiple Windows
- 34.9 Input and Commands in a Multi-Buffer Editor
- 34.10 File Management with Multiple Buffers
- 34.11 Complexity and Resource Costs
- 34.12 Design Alternatives and Trade-offs
- 34.13 Common Multiple-Buffer Mistakes

---

# **Volume VIII — Software Engineering**

## **Chapter 35 — Testing**

- 35.1 Why Testing Matters
- 35.2 Testing Strategy
- 35.3 Unit Testing
- 35.4 Testing the Document
- 35.5 Testing the Cursor
- 35.6 Testing the Viewport
- 35.7 Testing Input Parsing
- 35.8 Integration Testing
- 35.9 End-to-End Testing
- 35.10 Regression Testing
- 35.11 Testing Failure Cases
- 35.12 Test Organization
- 35.13 Common Testing Mistakes
- 35.14 Testing Invariants

## **Chapter 36 — Debugging**

- 36.1 Why Editors Are Difficult to Debug
- 36.2 Assertions
- 36.3 Debug Builds
- 36.4 Using a Debugger
- 36.5 Breakpoints
- 36.6 Inspecting Editor State
- 36.7 Debugging Input
- 36.8 Debugging Rendering
- 36.9 Logging and Tracing
- 36.10 Diagnosing Memory Errors
- 36.11 Reproducing Bugs
- 36.12 Common Debugging Mistakes

## **Chapter 37 — Portability**

- 37.1 Why Portability Matters
- 37.2 Platform Boundaries
- 37.3 POSIX and Platform APIs
- 37.4 Linux
- 37.5 macOS and BSD
- 37.6 Windows
- 37.7 WSL
- 37.8 Conditional Compilation
- 37.9 Common Portability Mistakes

## **Chapter 38 — Packaging**

- 38.1 Development Builds
- 38.2 Release Builds
- 38.3 CMake Build Configurations
- 38.4 Installation
- 38.5 Release Artifacts
- 38.6 Versioning
- 38.7 Documentation
- 38.8 Distribution
- 38.9 Reproducible Builds
- 38.10 Preparing the Editor for Release

---

# **Volume IX — Advanced Topics**

## **Chapter 39 — Plugin Architecture**

- 39.1 Why Plugins Change the Architecture
- 39.2 Plugin Boundaries
- 39.3 Designing a Plugin Interface
- 39.4 ABI — When Source Compatibility Is Not Enough
- 39.5 Plugin Versioning and Compatibility
- 39.6 Dynamic Loading
- 39.7 Extending the Editor Safely
- 39.8 Plugin Failure and Isolation
- 39.9 Security Considerations
- 39.10 A Minimal Plugin Architecture
- 39.11 Design Alternatives and Trade-offs
- 39.12 What Our Editor Would Need to Support Plugins

## **Chapter 40 — Language Server Protocol (LSP)**

- 40.1 Why Language Intelligence Matters
- 40.2 What Is the Language Server Protocol?
- 40.3 LSP Client and Language Server Architecture
- 40.4 Starting and Managing a Language Server
- 40.5 Document Synchronization
- 40.6 Diagnostics
- 40.7 Completion
- 40.8 Hover Information
- 40.9 Requests, Notifications, and Asynchronous Results
- 40.10 LSP Integration Trade-offs and Boundary for Our Editor

## **Chapter 41 — Tree-sitter and Incremental Parsing**

- 41.1 Why Incremental Parsing Matters
- 41.2 From Tokens to Syntax Trees
- 41.3 What "Incremental" Really Means
- 41.4 Tree-sitter as an Incremental Parser
- 41.5 Parsing an Edit
- 41.6 Syntax Tree Structure and Node Ranges
- 41.7 Concrete Syntax Trees vs Abstract Syntax Trees
- 41.8 Error Recovery and Incomplete Code
- 41.9 Incremental Syntax Highlighting
- 41.10 Beyond Highlighting - Structural Editor Features
- 41.11 Tree-sitter and the Language Server
- 41.12 Complexity and Performance
- 41.13 Tree-sitter vs the Chapter 32 Baseline
- 41.14 Why Our Baseline Does Not Adopt Tree-sitter
- 41.15 Common Tree-sitter Integration Mistakes
- 41.16 Design Trade-offs

## **Chapter 42 — Terminal Graphics**

- 42.1 The Terminal Model and Capability Surface
- 42.2 True Color
- 42.3 Terminal Capability Detection
- 42.4 Images in Terminals
- 42.5 The Kitty Graphics Protocol
- 42.6 Mouse Input
- 42.7 Mouse Coordinates and Editor Coordinates
- 42.8 Why Graphics Complicate Full Redraw
- 42.9 The Portability Cost of Advanced Terminal Features
- 42.10 Security and Resource Considerations
- 42.11 Decision and Architectural Lessons

## **Chapter 43 — Modern Text Editors: Architecture in the Real World**

- 43.1 Why Study Existing Editors?
- 43.2 How to Compare an Editor
- 43.3 Nano — Simplicity as an Architectural Choice
- 43.4 Vim — Modal Editing and Command-Centric Architecture
- 43.5 Neovim — Refactoring the Editor Around APIs
- 43.6 Emacs — The Editor as an Extensible Environment
- 43.7 Helix — A Modern Modal Architecture
- 43.8 VS Code — The Editor as a Platform
- 43.9 Sublime Text — Performance and Responsiveness
- 43.10 Zed — Modern Native Editor Architecture
- 43.11 Architecture Comparison
- 43.12 What Real Editors Teach Us
- 43.13 Our Architecture in Context
- 43.14 What We Are Not Building

---

# **Volume X — Building Our Editor**

This is the implementation volume.

## **Chapter 44 — Volume X Preparation**

- 44.1 From Theory to Implementation
- 44.2 Setting Up the Development Workspace
- 44.3 Creating the Implementation Repository
- 44.4 Initializing Git
- 44.5 Establishing the CMake Build
- 44.6 Compiler and Warning Policy
- 44.7 Development and Release Build Configurations
- 44.8 Establishing the Initial Project Structure
- 44.9 Establishing the Testing Baseline
- 44.10 Implementation Documentation and Status Tracking
- 44.11 The Implementation Workflow
- 44.12 Establishing the Initial Implementation Baseline

## **Chapter 45 — Establishing the Runtime**

- 45.1 Implementation Goal
- 45.2 Terminal Module
- 45.3 Saving and Restoring Terminal State
- 45.4 Enabling Raw Mode
- 45.5 Application Lifecycle
- 45.6 Minimal Event Loop
- 45.7 Manual Verification
- 45.8 Resulting Project State

## **Chapter 46 — Building the First Editable Screen**

- 46.1 Implementation Goal
- 46.2 Implementing the Document
- 46.3 Implementing the Cursor
- 46.4 Implementing Editor State
- 46.5 Implementing the Screen Buffer
- 46.6 Implementing the Renderer
- 46.7 Integrating the First Editable Screen
- 46.8 Implementing the First Interactive Editing Loop
- 46.9 Build, Runtime Verification, and Integration Review

## **Chapter 47 — Editing Text**

Character insertion
Character deletion
Cursor movement
Line boundaries
Dirty state
Basic editing commands

## **Chapter 48 — Working with Lines**

Line splitting
Line merging
End-of-line behavior
Cursor transitions
Editing invariants

## **Chapter 49 — Navigating the Document**

Vertical movement
Preferred column
Viewport
Vertical scrolling
Horizontal scrolling
Cursor visibility

## **Chapter 50 — Opening Files**

File Manager
File loading
UTF-8 decoding
Line-ending normalization
Empty files
Initial editor state

## **Chapter 51 — Saving Files**

Encoding
File serialization
Temporary files
Atomic replacement
Dirty flag
Save errors
Terminal/editor recovery

## **Chapter 52 — Completing the Editor Interface**

Input parsing
Command dispatch
Status Bar
Message Bar
Key bindings
Filename/cursor information
User-facing errors

## **Chapter 53 — Hardening the Editor**

Invariants
Boundary cases
Failure paths
Testing
Debugging
Sanitizers
Terminal restoration
Regression testing

## **Chapter 54 — Final Integration and Release**

Architecture review
Module cleanup
Portability checks
Build configurations
Packaging
Documentation
Final end-to-end validation

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
