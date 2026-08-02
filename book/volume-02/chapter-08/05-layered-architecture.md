# 8.5 Layered Architecture

In the previous section, we identified the major components of our editor and examined how they cooperate to process user input, update the document, and render the screen.

However, knowing the components alone is not enough. We must also decide **how these components are allowed to interact**.

If every component could communicate directly with every other component, the system would quickly become difficult to understand and maintain. As more features are added, the number of dependencies would grow rapidly, making even small changes risky.

To avoid this problem, software architects often organize systems into **layers**.

A **layered architecture** groups related components into levels, where each layer has a specific purpose and communicates only with the layers immediately above or below it.

This organization provides a clear structure that keeps responsibilities separated and dependencies manageable.

## What Is a Layer?

A layer is a collection of components that perform similar kinds of work.

Rather than viewing the application as one large collection of interacting modules, we divide it into levels of abstraction.

Higher layers focus on **application behavior**, while lower layers deal with **system-specific details**.

A simple layered architecture can be visualized as follows:

```text
+----------------------+
| Application Layer    |
+----------------------+
           │
           ▼
+----------------------+
| Core Logic Layer     |
+----------------------+
           │
           ▼
+----------------------+
| Platform Layer       |
+----------------------+
```

Each layer builds upon the services provided by the layer beneath it.

The application layer does not need to know how terminal input is read.

Likewise, the platform layer does not need to understand editing commands.

This separation keeps each layer focused on its own responsibilities.

## Why Use Layers?

Layering is one of the most common architectural patterns because it provides several important advantages.

### Separation of Responsibilities

Each layer performs a different kind of work.

For example:

- One layer handles user interaction.
- Another manages the editor's internal state.
- Another communicates with the operating system.

Because these responsibilities are separated, the implementation of one layer remains largely independent of the others.

### Reduced Coupling

Without layers, components might communicate freely.

```text
Renderer ─────────► File Manager
     │                  ▲
     │                  │
     ▼                  │
Cursor ◄────────► Input Handler
        ▲
        │
Document
```

As the number of connections increases, understanding the system becomes increasingly difficult.

Layering restricts these interactions.

Instead of communicating arbitrarily, each component follows well-defined paths through the architecture.

This greatly simplifies the dependency structure.

### Easier Maintenance

Suppose we later decide to support a different terminal interface.

Ideally, only the platform-specific code should change.

The editing logic should continue to operate exactly as before.

Layering makes this possible because higher-level components depend on abstract behavior rather than platform-specific implementation details.

Similarly, improvements to the renderer should not require changes to file handling or command processing.

### Better Reusability

Components located in lower layers often provide general services that can be reused by multiple higher-level components.

For example, both the renderer and the file manager may need access to the current document stored in the Editor State.

Rather than maintaining separate copies, both components simply access the shared state provided by the core layer.

## Layers in Our Editor

Our terminal text editor can be organized into three logical layers.

```text
+--------------------------------------+
|          Application Layer           |
|--------------------------------------|
| Command Engine                       |
+--------------------------------------+
                 │
                 ▼
+--------------------------------------+
|            Core Layer                |
|--------------------------------------|
| Editor State                         |
| Document                             |
| Cursor                               |
| Viewport                             |
+--------------------------------------+
                 │
                 ▼
+--------------------------------------+
|          Platform Layer              |
|--------------------------------------|
| Input Handler                        |
| Input Parser                         |
| Renderer                             |
| File Manager                         |
| Terminal Interface                   |
+--------------------------------------+
```

Each layer serves a distinct purpose within the overall system.

## The Application Layer

The **Application Layer** represents the editor's behavior.

Its primary responsibility is deciding **what should happen** in response to user actions.

The central component of this layer is the **Command Engine**.

When the user presses a key, the Command Engine determines which editing operation should be performed.

Examples include:

- Insert a character.
- Delete text.
- Move the cursor.
- Save the document.
- Exit the editor.

The Application Layer coordinates operations but does not concern itself with low-level implementation details such as reading keyboard input or drawing the screen.

## The Core Layer

The **Core Layer** represents the editor itself.

It contains the data that defines the current editing session.

This layer includes components such as:

- Editor State
- Document
- Cursor
- Viewport

Together, these components represent the complete state of the application at any given moment.

One of the defining characteristics of the Core Layer is that it remains largely independent of the operating system.

Whether the editor runs inside Linux, macOS, or another POSIX-compatible environment, the concepts of a document, cursor, and viewport remain the same.

For this reason, the Core Layer forms the heart of the application.

## The Platform Layer

The **Platform Layer** interacts directly with the external environment.

It communicates with:

- The terminal
- The operating system
- The file system

This layer includes components such as:

- Input Handler
- Input Parser
- Renderer
- File Manager
- Terminal Interface

Unlike the Core Layer, these components depend on platform-specific behavior.

For example:

- The Input Handler uses POSIX terminal APIs to read keyboard input.
- The Renderer sends ANSI escape sequences to the terminal.
- The File Manager performs operating system file operations.

By isolating these responsibilities within the Platform Layer, the remainder of the editor remains largely unaffected by platform-specific implementation details.

## Dependency Direction

One of the most important rules of layered architecture is that **dependencies should flow in one direction**.

Higher layers may depend on lower layers, but lower layers should not depend on higher layers.

This relationship can be visualized as follows.

```text
Application Layer
        │
        ▼
Core Layer
        │
        ▼
Platform Layer
```

Notice that the arrows point downward.

The Application Layer uses services provided by the Core Layer.

The Platform Layer provides services to the rest of the application but should not contain knowledge of application-specific behavior.

Maintaining this one-way dependency structure prevents circular relationships between modules and makes the architecture easier to understand.

## Layers Are Logical, Not Physical

It is important to understand that architectural layers are **logical groupings**, not necessarily directories or source files.

A project may contain dozens of files implementing the Core Layer or only a few.

Similarly, several components may reside in the same directory while still belonging to different architectural layers.

Layers describe **responsibilities**, not file organization.

The physical project structure will be discussed later in Chapter 10.

## Engineering Insight

Many modern software systems—web applications, operating systems, databases, and compilers—are organized using layered architectures. Although the number and names of the layers vary, the underlying idea remains the same: separate high-level application behavior from low-level implementation details.

Our editor follows this proven approach on a smaller scale. By introducing layers now, we establish a clear boundary between _what_ the editor does, _how_ it manages its state, and _how_ it interacts with the operating system. This organization will make the implementation in later chapters significantly easier to follow.

## Summary

A layered architecture organizes a software system into levels of responsibility, each performing a distinct role while communicating through well-defined dependency paths. This approach reduces coupling, improves maintainability, and creates a clearer separation between application logic, core editor state, and platform-specific services.

For our terminal text editor, the architecture consists of an **Application Layer**, a **Core Layer**, and a **Platform Layer**. Together, these layers provide a structured foundation that supports the modular design introduced earlier in the chapter.

In the next section, we will examine each of the editor's major components individually, defining their responsibilities and explaining how they contribute to the overall architecture.
