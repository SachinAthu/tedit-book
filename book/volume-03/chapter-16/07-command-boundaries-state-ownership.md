# 16.7 Command Boundaries and State Ownership

The Command Engine coordinates changes to the editor, but it does not own every piece of state involved in those changes.

This distinction becomes important as commands begin to affect multiple components. A single command may involve the Document, Cursor, Viewport, Dirty Flag, or other parts of the Editor State. Without clear ownership rules, the Command Engine could gradually become responsible for all of them, turning it into a large and tightly coupled component.

The architecture avoids this by separating **command execution** from **state ownership**.

```text
Command Engine
      │
      │ coordinates
      ▼
+-------------------------------+
|         Editor State          |
|                               |
|  Document   Cursor   Viewport |
|  File       Dirty    Screen   |
|  Status     Configuration     |
+-------------------------------+
      │
      │ owned by individual
      │ core components
      ▼
Core Components
```

The Command Engine determines **which state needs to change** as part of a command, while the component responsible for that state applies the appropriate rules.

## Command Execution Is Not State Ownership

Consider a cursor movement command:

```text
MOVE_CURSOR_LEFT
       │
       ▼
Command Engine
       │
       ▼
Cursor
```

The Command Engine executes the command, but it does not become the owner of the cursor's internal rules.

The Cursor remains responsible for determining how the movement should occur.

Similarly:

```text
INSERT_CHARACTER('A')
       │
       ▼
Command Engine
       │
       ├──► Document
       └──► Cursor
```

The Command Engine coordinates the operation, but the Document remains responsible for its text representation and the Cursor remains responsible for cursor behavior.

This preserves the principle established by the architecture:

> **One owner, many readers.**

## The Command Engine as a Coordinator

The Command Engine can therefore be viewed as a coordinator sitting above several core components:

```text
                    Command
                       │
                       ▼
               +---------------+
               | Command Engine|
               +---------------+
                  │    │    │
                  │    │    │
                  ▼    ▼    ▼
             Document Cursor File
                  │    │    │
                  └────┼────┘
                       ▼
                 Editor State
```

Its responsibility is to coordinate the sequence of operations required to execute a command.

For example, inserting a character may require:

1. inserting the character into the Document,
2. updating the Cursor,
3. marking the Document as modified.

The Command Engine coordinates those effects, but it should not independently implement the internal logic of each component.

## State Ownership

Each important piece of state should have a clear owner.

The architecture defines ownership explicitly:

| State          | Owner                        |
| -------------- | ---------------------------- |
| Document       | Document / editing component |
| Cursor         | Cursor                       |
| Viewport       | Command Engine               |
| Current File   | File Manager                 |
| Dirty Flag     | Command Engine               |
| Status Message | Renderer                     |
| Screen Size    | Terminal Interface           |
| Configuration  | Command Engine               |

The exact ownership boundaries matter because they determine which component is allowed to make authoritative changes.

For example, the Cursor component should not silently modify the Document. The Renderer should not modify the Cursor simply because it is drawing it. The File Manager should not become responsible for rendering a save message.

Each component has a defined responsibility, and the Command Engine coordinates those components when a command requires several of them to participate.

## Why Ownership Matters

Without explicit ownership, state can become duplicated.

Imagine keeping the cursor position in both the Command Engine and the Cursor component:

```text
Command Engine
    cursor_row = 5

Cursor
    cursor_row = 5
```

After one operation, the two values could diverge:

```text
Command Engine
    cursor_row = 5

Cursor
    cursor_row = 6
```

Now the system has two conflicting versions of the same state.

A single owner eliminates this ambiguity:

```text
Cursor
    │
    └── cursor position
```

Other components read the value when they need it, but the Cursor remains the authoritative owner.

## Coordination Without Duplication

The same principle applies to the Document.

The Command Engine may execute:

```text
INSERT_CHARACTER('A')
```

but it should not maintain another copy of the text to track what happened.

Instead:

```text
Command Engine
      │
      │ request operation
      ▼
Document
      │
      │ authoritative text
      ▼
Editor State
```

The Command Engine needs access to the Document, but it does not need to duplicate the Document's data.

This keeps the architecture consistent with the single-source-of-truth principle established for Editor State.

## Commands Can Cross Ownership Boundaries

A command can legitimately involve multiple owners.

For example:

```text
INSERT_CHARACTER('A')
       │
       ▼
Command Engine
       │
       ├───────────────► Document
       │                     │
       │                     └── text changes
       │
       ├───────────────► Cursor
       │                     │
       │                     └── position changes
       │
       └───────────────► Dirty State
                             │
                             └── modified
```

This does not mean that the Command Engine owns all three pieces of state.

It means that the command requires a coordinated transition involving several state owners.

This distinction keeps the Command Engine from becoming a general-purpose state container.

## Renderer Ownership Is Different

The Renderer provides an especially useful example of why ownership boundaries matter.

The Renderer reads the Editor State:

```text
Editor State
      │
      ▼
Renderer
      │
      ▼
Terminal
```

It may calculate visual positions and generate a frame, but it should not modify the underlying Document or Cursor.

For example, the Renderer may determine where the cursor should appear on screen, but that does not make it the owner of the logical cursor position.

The logical position belongs to the Cursor. The visual representation belongs to the Renderer.

This distinction allows rendering to remain a read-only operation over the current state, consistent with the full-redraw architecture.

## File Operations and Ownership

The same boundary applies when executing `SAVE`.

The Command Engine initiates the command:

```text
SAVE
 │
 ▼
Command Engine
 │
 ▼
File Manager
```

The File Manager handles the interaction with the file system, while the Editor State continues to contain the current in-memory Document.

Conceptually:

```text
                SAVE
                  │
                  ▼
           Command Engine
                  │
                  ▼
             File Manager
              │        │
              │        └── File System
              │
              └── Editor State
```

The Command Engine therefore coordinates the operation without becoming responsible for low-level file I/O.

Detailed file ownership and persistence behavior will be addressed later in the File System volume.

## A Useful Architectural Rule

A useful rule for the implementation is:

> **The Command Engine coordinates state changes; the component that owns a piece of state remains responsible for the rules governing that state.**

This can be expressed as:

```text
Command Engine
      │
      │ coordinates
      ▼
State Owner
      │
      │ applies rules
      ▼
Updated State
```

This rule prevents command execution from becoming a collection of unrelated algorithms.

It also makes the architecture easier to reason about. When a bug occurs, we can ask two separate questions:

1. **Was the correct command executed?**
2. **Did the responsible component apply its state rules correctly?**

Those are different problems and can be investigated independently.

## The Boundary in the Overall Architecture

The complete relationship can now be viewed as:

```text
Input Event
     │
     ▼
Input Engine
     │
     │ creates
     ▼
Command
     │
     ▼
Command Engine
     │
     │ coordinates
     ▼
+----------------------+
|     Core State       |
|                      |
| Document             |
| Cursor               |
| Viewport             |
| Dirty Flag           |
+----------------------+
     │
     │ read
     ▼
Renderer
```

The Command Engine is therefore neither an input component nor a data-storage component.

It is the **behavioral coordinator** between the two.

This boundary is what allows the architecture to remain modular:

- Input components determine **what the user requested**.
- The Command Engine determines **which operations must occur**.
- Core components determine **how their state is changed correctly**.
- The Renderer determines **how the resulting state is displayed**.

Keeping these responsibilities separate prevents the Command Engine from becoming a central module that knows the internal details of every other component.

## Boundary Contract

The Command Engine's relationship with state ownership can be summarized as follows:

| Responsibility                   | Command Engine |
| -------------------------------- | -------------- |
| Receive commands                 | Yes            |
| Decide which command to execute  | Yes            |
| Coordinate state changes         | Yes            |
| Own Document representation      | No             |
| Define cursor movement rules     | No             |
| Render state                     | No             |
| Perform terminal rendering       | No             |
| Perform low-level file I/O       | No             |
| Duplicate core state             | No             |
| Coordinate multiple state owners | Yes            |

The resulting principle is simple:

```text
                Command
                   │
                   ▼
           Command Engine
                   │
             coordinates
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
    Document     Cursor     File
     owns         owns      owns
     text        position   I/O
```

The Command Engine gives these components a place to cooperate without taking ownership of their responsibilities.

That separation will become increasingly important as more commands are introduced. The goal is not to make the Command Engine responsible for everything the editor can do, but to make it the clear and predictable place where **editor intent is translated into coordinated state transitions**.

## Summary

The Command Engine coordinates state changes but does not own the state involved in those changes. Command execution is not state ownership: each piece of state has a single owner, and the Command Engine directs which owners must participate in a command's transition. A command may cross ownership boundaries without turning the Command Engine into a general-purpose state container.

This rule prevents duplicated state, keeps the Renderer read-only over the Editor State, and makes bugs easier to isolate. The Command Engine is the **behavioral coordinator** between input and core state.

Now that the ownership boundaries are clear, the next section examines commands that do not modify the Document.

Continue to **Section 16.8, Commands That Do Not Modify the Document**.
