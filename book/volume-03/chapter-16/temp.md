# Chapter 16 — Command Engine

# 16.2 What the Command Engine Is Responsible For

Define the Command Engine's responsibilities without yet explaining individual commands.

Responsibilities:

- Receive commands from the Input Engine
- Execute the requested operation
- Coordinate changes to Editor State
- Invoke the appropriate core operation
- Maintain command-level invariants
- Report command results where necessary
- Determine whether a command changes state

And explicitly define what it **does not** do:

- Does not read terminal bytes
- Does not parse escape sequences
- Does not decide which key means which command
- Does not render the screen
- Does not directly handle terminal control
- Does not own document data

This is important because the architecture already establishes these boundaries.

### No duplication

This section should **not re-explain**:

- Input Handler → Chapter 6 / earlier foundations
- Input Parser → Chapter 6 and Chapter 15
- Editor State → Chapter 12
- Document responsibility → Chapter 13
- Cursor rules → Chapter 14

Instead, it should briefly reference those concepts.

---

# 16.3 Commands as an Abstraction

This should be one of the major conceptual sections.

Explain why commands exist as a separate concept.

For example:

```text
KEY_CTRL_S
      │
      ▼
SAVE_DOCUMENT
```

versus:

```text
KEY_ARROW_LEFT
      │
      ▼
MOVE_CURSOR_LEFT
```

The important observation is that the command describes **intent**, not input.

A command should therefore be independent of the key that triggered it.

For example:

```text
KEY_ARROW_LEFT  ─────► COMMAND_MOVE_LEFT
KEY_CTRL_H      ─────► COMMAND_MOVE_LEFT
```

Both could theoretically invoke the same command.

This naturally reinforces the conclusion of ADR-015: key bindings are relationships between input events and commands, while commands represent editor behavior.

### This is new material

It wasn't fully developed in Chapter 15, so this deserves substantial treatment here.

---

# 16.4 Command Representation

Now introduce how commands can be represented in C.

For example, conceptually:

```text
Command

├── Move Left
├── Move Right
├── Move Up
├── Move Down
├── Insert Character
├── Delete
├── Split Line
├── Save
└── Quit
```

Then discuss an enum such as:

```text
CommandType
```

and, where appropriate, command data.

For example, an insertion command needs additional information:

```text
COMMAND_INSERT_CHARACTER
        │
        └── 'A'
```

Whereas:

```text
COMMAND_MOVE_LEFT
```

requires no additional payload.

This section should establish the distinction between:

**Command type**

and

**Command data**

without getting into implementation details yet.

---

# 16.5 The Command Execution Lifecycle

This should be the chapter's central process section.

Show:

```text
Input Event
     │
     ▼
Input Engine
     │
     ▼
Command
     │
     ▼
Command Engine
     │
     ├── Validate
     │
     ├── Execute
     │
     ├── Update State
     │
     └── Report Result
     │
     ▼
Editor State
     │
     ▼
Renderer
```

Then connect it to the runtime model:

```text
Wait
 ↓
Parse
 ↓
Map
 ↓
Execute
 ↓
Update
 ↓
Render
```

The runtime model already defines this pipeline, so **we should not teach the Event Loop again here**. Chapter 16 should zoom in specifically on the **Execute Command** stage.

---

# 16.6 Command Categories

Instead of immediately having seven independent sections, classify commands by what they affect.

### Navigation commands

```text
MOVE_LEFT
MOVE_RIGHT
MOVE_UP
MOVE_DOWN
```

### Editing commands

```text
INSERT_CHARACTER
DELETE_CHARACTER
SPLIT_LINE
MERGE_LINE
```

### Document commands

```text
SAVE
```

### Application commands

```text
QUIT
```

This classification gives the reader a useful mental model without repeating the detailed behavior already covered elsewhere.

---

# 16.7 Editing Commands and Core Operations

This is where we connect commands to the previous chapters.

The key idea:

> **The Command Engine coordinates operations; it does not redefine their rules.**

For example:

```text
COMMAND_DELETE
      │
      ▼
Document operation
      │
      ▼
Cursor adjustment
      │
      ▼
Editor State
```

Then briefly show the relationships for:

- Insert
- Delete
- Split line
- Merge line
- Move cursor

But **do not re-teach how these operations work**.

For example, Chapter 13 already establishes the document representation as a vector of lines. Chapter 14 already establishes cursor movement rules and preferred-column behavior.

So this section should answer:

> **Which subsystem does the command need to coordinate?**

rather than:

> How does insertion/deletion/cursor movement work?

That distinction will eliminate a lot of duplication.

---

# 16.8 Command Boundaries and State Ownership

This section is important because the architecture documents contain a potentially confusing point: the Command Engine coordinates state changes, while individual components such as Cursor and Document own particular state.

We should clarify the conceptual distinction:

```text
Command Engine
    │
    │ coordinates
    ▼
Core Components
    │
    ├── Document
    ├── Cursor
    └── Viewport
```

The Command Engine shouldn't become a giant object that directly owns every field.

Instead:

> **The Command Engine owns command execution, while core components own the state they are responsible for.**

This aligns with the broader “one owner, many readers” principle from the architecture.

This section will also prevent a future implementation mistake where `command.c` becomes a giant collection of low-level document and cursor algorithms.

---

# 16.9 Commands That Do Not Modify the Document

This is worth separating because not every command is an editing command.

For example:

```text
MOVE_CURSOR_LEFT
```

changes editor state but not document contents.

```text
SAVE
```

does not modify the document at all; it synchronizes in-memory state with the filesystem.

```text
QUIT
```

changes application lifecycle state.

This gives us a useful classification:

| Command    | Document | Cursor | Other State |
| ---------- | -------: | -----: | ----------- |
| Move Left  |        — |      ✓ |             |
| Insert     |        ✓ |      ✓ | Dirty       |
| Delete     |        ✓ |      ✓ | Dirty       |
| Split Line |        ✓ |      ✓ | Dirty       |
| Save       |        — |      — | Dirty       |
| Quit       |        — |      — | Running     |

This is a **new architectural view**, rather than a repetition of the operation rules.

---

# 16.10 Command Results and Error Handling

Some commands succeed; others may fail.

For example:

```text
SAVE
  │
  ├── success
  │
  └── failure
```

A save failure should not necessarily terminate the editor.

Instead:

```text
Command
   │
   ▼
Result
   │
   ├── Success
   │
   └── Error
          │
          ▼
     Message State
```

This connects to the runtime model's distinction between recoverable and fatal errors.

Again, we don't need to repeat the entire error-handling architecture—just explain what it means specifically at the command boundary.

---

# 16.11 Command Execution and the Dirty State

This deserves a short section because it is an important consequence of command execution.

Conceptually:

```text
Editing Command
      │
      ▼
Document changes
      │
      ▼
Dirty = true
```

while:

```text
Save
  │
  ▼
File successfully written
  │
  ▼
Dirty = false
```

The important point is that the **Command Engine knows that commands have state consequences**, even though the actual document data belongs to the Document.

This will also prepare readers for the later File I/O chapter without pulling file-system implementation details into Chapter 16.

---

# 16.12 Command Dispatch

Now we can discuss the actual execution mechanism.

Conceptually:

```text
switch (command.type)
{
    case COMMAND_INSERT:
        ...
        break;

    case COMMAND_DELETE:
        ...
        break;

    case COMMAND_MOVE_LEFT:
        ...
        break;

    ...
}
```

Discuss why a simple dispatch mechanism is appropriate for this project.

Possible alternatives:

- `switch` dispatch
- Function-pointer command table
- Command objects
- Command pattern

But don't over-engineer it.

The project prioritizes simplicity and educational value, and the architecture intentionally keeps the command boundary explicit.

---

# 16.13 Design Alternatives

This should follow the same architecture-first style as previous chapters.

### Alternative 1 — Input Engine executes commands

```text
Input Engine
      │
      ▼
State
```

Problem:

The Input Engine would become responsible for both mapping input and executing editor behavior.

---

### Alternative 2 — Command Engine merged into Input Engine

```text
Input
  │
  ▼
Input Engine
  │
  ▼
State
```

This is simpler for a tiny editor but creates a larger responsibility boundary.

---

### Alternative 3 — Commands directly modify individual modules

Potentially:

```text
Command
 ├── Document
 ├── Cursor
 └── File
```

Discuss the coupling implications.

---

### Alternative 4 — Full Command Pattern / command objects

Useful for future undo/redo, but unnecessary for the current baseline.

This can become a valuable forward reference to Chapter 32.

---

# 16.14 Complexity and Trade-offs

Discuss the cost of the abstraction itself.

For example:

| Aspect                    | Impact            |
| ------------------------- | ----------------- |
| Runtime overhead          | Very small        |
| Memory overhead           | Small             |
| Code structure            | More explicit     |
| Testability               | Improved          |
| Input independence        | Improved          |
| Future key remapping      | Easier            |
| Undo/redo potential       | Better foundation |
| Implementation complexity | Slightly higher   |

The important conclusion should be:

> The Command Engine does not primarily exist for performance. It exists to establish a clean boundary between **input intent** and **editor behavior**.

---

# What I would deliberately NOT include

This is the most important part for avoiding duplication.

### ❌ Don't re-explain cursor movement

Chapter 14 already defines:

- Left/Right
- Up/Down
- line boundaries
- preferred column
- cursor constraints

Chapter 16 should only say:

> `MOVE_LEFT` invokes the cursor's left-movement operation.

The actual rules stay in Chapter 14.

### ❌ Don't re-explain document editing algorithms

Chapter 13 establishes the Document and its representation. Chapter 16 shouldn't teach how vector insertion, deletion, or line splitting works.

### ❌ Don't re-explain input parsing

Chapter 15 has just established:

```text
Raw Input
    ↓
Input Parser
    ↓
Key Event
```

Chapter 16 starts at the **command** boundary.

### ❌ Don't re-explain the Event Loop

Chapter 9 and the Runtime Model already establish:

```text
Parse
 ↓
Execute
 ↓
Update
 ↓
Render
```

Chapter 16 should zoom into **Execute**, not teach the whole loop again.

### ❌ Don't teach rendering

Rendering belongs to Volume IV, beginning with Chapter 17.

### ❌ Don't teach file I/O

`SAVE` can be used as an example of a command that invokes file functionality, but actual file loading, saving, permissions, atomic writes, etc. belong to Chapters 22–24.

### ❌ Don't teach undo/redo

The command abstraction can be identified as a **possible foundation** for future undo/redo, but the actual Command Pattern and undo/redo architecture belongs to Chapter 32.

---

# Final Chapter 16 structure

So I would settle on this:

1. **Introduction**
2. **What the Command Engine Is Responsible For**
3. **Commands as an Abstraction**
4. **Command Representation**
5. **The Command Execution Lifecycle**
6. **Command Categories**
7. **Editing Commands and Core Operations**
8. **Command Boundaries and State Ownership**
9. **Commands That Do Not Modify the Document**
10. **Command Results and Error Handling**
11. **Command Execution and the Dirty State**
12. **Command Dispatch**
13. **Design Alternatives**
14. **Complexity and Trade-offs**
15. **Common Mistakes**
16. **Command Engine Contract**
17. **Summary**
18. **Exercises**
19. **Preview of Chapter 17**

I think this is a much stronger Chapter 16 than simply having **Insert → Delete → Split Line → Merge Line → Move Cursor → Save → Quit** as seven feature sections. That older structure would inevitably duplicate Chapters 13 and 14.

More importantly, the progression across Chapters 12–16 becomes very clean:

```text
Chapter 12 — Editor State
        │
        │ What exists?
        ▼
Chapter 13 — Document Model
        │
        │ How is text represented?
        ▼
Chapter 14 — Cursor Engine
        │
        │ How is navigation represented?
        ▼
Chapter 15 — Input Engine
        │
        │ What does the user input mean?
        ▼
Chapter 16 — Command Engine
        │
        │ What should the editor do?
        ▼
Chapter 17 — Rendering Theory
        │
        │ How does state become a screen?
        ▼
        Renderer
```

That gives each chapter a **distinct architectural question**, which is the best protection against content duplication in the rest of the book. This also stays consistent with the project's stated philosophy of defining responsibilities and boundaries before implementation.

One small correction to the existing outline: **“Qui” should be “Quit.”** The SRS and runtime model both define exiting the application as a core behavior.
