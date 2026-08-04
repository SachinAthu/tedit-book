# 10.2 Mapping Architecture to Source Code

In Chapter 8, we designed the editor from an architectural perspective. We identified the major components of the system, defined their responsibilities, and established the relationships between them. However, an architecture diagram is a logical model—it describes **what** components exist and **how** they interact, but it does not specify how those components should be organized within a software project.

The next step is to translate that logical architecture into a physical organization of source code.

This process is known as **mapping architecture to source code**. Each architectural component becomes one or more modules in the project, with its own files, public interface, and implementation. The goal is for someone looking at the project's directory structure to recognize the architecture immediately.

For example, the architecture introduced in Chapter 8 contains components such as the Input Handler, Renderer, Document, and File Manager. Rather than combining all of their functionality into a few large source files, we implement each responsibility as a separate module.

```text
Architecture                  Source Code

Input Handler        ─────►    input/

Renderer             ─────►    renderer/

Document             ─────►    document/

Cursor               ─────►    cursor/

File Manager         ─────►    file/

Command Engine       ─────►    command/

Terminal Interface   ─────►    terminal/
```

Notice that the names of the modules closely match the names of the architectural components. This is intentional. Using consistent terminology throughout the architecture, documentation, and source code reduces confusion and makes the project easier to understand.

## From Components to Modules

An architectural component represents a responsibility within the system. A module is the implementation of that responsibility.

For example, consider the Renderer.

In the architecture, the Renderer is responsible for generating the editor's visual output from the current Editor State.

When implemented, this responsibility becomes a dedicated module:

```text
renderer/

├── renderer.h
└── renderer.c
```

The header file declares the services the Renderer provides, while the source file contains the implementation. Other parts of the editor interact with the Renderer through its public interface without needing to understand its internal implementation.

The same principle applies to every major architectural component.

## Preserving Architectural Boundaries

One of the primary goals of project organization is to preserve the boundaries established by the architecture.

Suppose the architecture specifies that the Renderer is responsible only for drawing the screen. If rendering code becomes mixed with document editing or file handling logic, the implementation no longer reflects the architecture. Over time, the boundaries between components become blurred, making the system more difficult to understand and maintain.

Instead, each module should remain focused on the responsibility assigned to it during the architectural design phase.

```text
+------------------+
|    Renderer      |
+------------------+
| Draw screen      |
| Draw status bar  |
| Draw cursor      |
+------------------+
```

It should **not** perform tasks such as:

- Reading keyboard input
- Editing document contents
- Saving files
- Parsing commands

Keeping these responsibilities separate ensures that the implementation remains faithful to the architecture.

## One Responsibility, One Home

A useful guideline when organizing a project is that every significant responsibility should have a single, obvious home.

If a developer wants to understand how cursor movement works, they should naturally look in the Cursor module. If they need to modify file loading, they should know to examine the File Manager.

This predictability greatly improves navigation through the codebase. Developers spend less time searching for functionality because the project structure mirrors the mental model established by the architecture.

## Architecture as a Blueprint

An architectural diagram should be viewed as a blueprint rather than an abstract drawing.

Just as an architect's blueprint identifies where each room belongs before construction begins, a software architecture identifies where each responsibility belongs before implementation starts.

The implementation phase then follows that blueprint by creating modules that correspond to the architectural components.

```text
Architecture
      │
      ▼
Responsibilities
      │
      ▼
Modules
      │
      ▼
Header Files
      │
      ▼
Source Files
```

By following this progression, the source code becomes a direct representation of the system's design rather than a collection of unrelated implementation files.

## Engineering Insight

A common mistake in small projects is to ignore architecture once implementation begins. Developers may design a clean architecture on paper but gradually place unrelated functionality wherever it is convenient. This process, often called **architectural erosion**, causes the implementation to drift away from the original design.

Mapping architectural components directly to source code modules helps prevent this problem. When the directory structure reflects the architecture, it becomes much easier to preserve module boundaries and maintain a clean, understandable codebase as the project evolves.

## Summary

Software architecture defines the logical organization of a system, while project structure defines how that organization is implemented in source code. By mapping each architectural component to a dedicated module, we preserve the design established in earlier chapters and create a codebase that is easier to navigate, maintain, and extend. In the next section, we will design the complete directory layout for our editor project and examine the purpose of each part of the project tree.
