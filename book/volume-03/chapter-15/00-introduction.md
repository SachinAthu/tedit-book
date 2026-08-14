# Volume III — Core Editor Engine

# Chapter 15 — Input Engine

## Introduction

A terminal text editor receives input as a continuous stream of keyboard events, but the rest of the editor should not need to understand how those events were produced.

Earlier chapters established how terminals deliver keyboard input and how raw bytes can be interpreted as meaningful key events. At that point, the low-level terminal details have served their purpose. The editor can now work with abstractions such as `KEY_ARROW_LEFT`, `KEY_BACKSPACE`, `KEY_ENTER`, and `KEY_CTRL_S` rather than dealing with bytes and escape sequences directly.

The next problem is deciding what those events mean to the editor.

For example, when the Input Parser produces:

```text
KEY_ARROW_LEFT
```

the editor needs to interpret that event as a request to move the cursor left. When it receives:

```text
KEY_CTRL_S
```

it needs to interpret it as a request to save the current document.

This creates an important boundary:

```text
Terminal Input
      │
      ▼
Input Handler
      │
      ▼
Input Parser
      │
      ▼
Key Event
      │
      ▼
Input Engine
      │
      ▼
Command
      │
      ▼
Command Engine
```

The **Input Engine** sits between input representation and editor behavior. Its responsibility is to take an already-parsed key event, determine which editor command it represents, and dispatch that command for execution.

This distinction is important. The Input Engine does not move the cursor, modify the document, save a file, or render the screen. Those operations belong to other parts of the architecture. Instead, it answers a narrower question:

> **What should this input request the editor to do?**

For example:

```text
KEY_ARROW_LEFT  →  MOVE_CURSOR_LEFT
KEY_BACKSPACE   →  DELETE_PREVIOUS_CHARACTER
KEY_ENTER       →  SPLIT_LINE
KEY_CTRL_S      →  SAVE_DOCUMENT
KEY_CTRL_Q      →  QUIT_EDITOR
```

The actual implementation of these commands will be the subject of the next chapter.

This separation gives the editor a clean flow from physical input to application behavior:

```text
Raw Bytes
    │
    ▼
Parsed Key Event
    │
    ▼
Input Mapping
    │
    ▼
Editor Command
    │
    ▼
Command Execution
    │
    ▼
Editor State
```

The architecture defined earlier already establishes this separation between input processing, command execution, and editor state. The Input Engine makes that boundary concrete without introducing another source of editor state or duplicating responsibilities that belong to the Command Engine.

The goal of this chapter is therefore not to teach keyboard input again. Instead, it is to understand how a clean editor turns **what the user pressed** into **what the editor should attempt to do**.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain the responsibility of the Input Engine within the editor architecture.
- Distinguish a **key event** from an **editor command**.
- Explain how parsed key events are mapped to editor commands.
- Understand how keyboard shortcuts are dispatched without coupling input handling to editor operations.
- Identify appropriate responsibilities for the Input Engine and responsibilities that belong to the Command Engine.
- Compare simple command-dispatch approaches such as conditional logic, `switch` statements, and command tables.
- Explain how unrecognized or unsupported input should be handled.
- Describe the complete flow from a parsed key event to command execution.
- Define a clear contract between the Input Engine and the Command Engine.
- Recognize common architectural mistakes that mix input processing with document editing, cursor management, rendering, or file operations.

By the end of the chapter, the input path should be conceptually clear:

```text
Key Event
    ↓
Input Engine
    ↓
Command
    ↓
Command Engine
```

The following chapter will build on this boundary by examining what happens when those commands are actually executed.
