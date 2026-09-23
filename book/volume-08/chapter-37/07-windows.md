# 37.7 Windows

Windows represents the largest platform difference considered in this chapter.

Linux, macOS, and BSD share a substantial POSIX foundation. Windows does not provide the same Unix/POSIX environment as its native system model. Consequently, supporting Windows requires a clearer separation between **portable editor behavior** and **platform-specific implementation**.

The important architectural question is therefore not:

> How can we make Windows behave exactly like Linux?

It is:

> **How can the same editor architecture be implemented on a fundamentally different platform?**

## Windows Changes the Portability Problem

The platform model can now be expanded:

```text
                     Editor
                       |
                       v
              Platform Boundary
                       |
          +------------+------------+
          |            |            |
        POSIX        POSIX        Windows
          |            |            |
        Linux        macOS         Native
                     BSD           APIs
```

Linux, macOS, and BSD can share substantial implementation because of their Unix/POSIX foundations.

Windows may require different mechanisms for:

- terminal input,
- terminal output,
- terminal mode configuration,
- filesystem operations,
- system handles,
- console interaction.

The editor's higher-level behavior should nevertheless remain the same.

## What Should Stay the Same?

The existence of a different operating system does not change the meaning of the editor.

The following remain application concepts:

```text
Document
Cursor
Viewport
Editor State
Command Engine
Undo/Redo
Search
Buffers
Windows
Rendering decisions
```

For example:

```text
Move Cursor Up
```

means the same thing regardless of whether the editor is running on Linux or Windows.

Likewise:

```text
Save Document
```

remains an editor command.

The implementation beneath these operations may differ.

```text
             Save Document
                   |
                   v
            File Manager
             /        \
            /          \
        POSIX          Windows
          |              |
       Linux/BSD/       Windows
       macOS
```

This distinction prevents Windows support from becoming a second editor implementation.

## Windows Does Not Require a Second Core

A common mistake when adding a substantially different platform is to duplicate the entire application.

That would lead toward:

```text
Linux Editor
├── Document
├── Cursor
├── Commands
├── Renderer
└── ...

Windows Editor
├── Document
├── Cursor
├── Commands
├── Renderer
└── ...
```

This duplicates the editor's semantics.

A better architecture is:

```text
                 Editor Core
                     |
          +----------+----------+
          |                     |
    POSIX Platform        Windows Platform
          |                     |
   Linux/macOS/BSD          Windows APIs
```

The Core remains shared.

Only the mechanisms required to interact with the environment differ.

This follows the architecture's existing separation between the Core Layer and Platform Layer.

## Native Windows APIs

Windows provides its own native system interfaces rather than relying exclusively on POSIX APIs.

Consequently, a Windows implementation may need platform-specific code for operations that the POSIX implementation handles through Unix-oriented interfaces.

Conceptually:

```text
POSIX implementation:

File Manager
     |
     v
POSIX file interface
     |
     v
Operating System


Windows implementation:

File Manager
     |
     v
Windows file interface
     |
     v
Operating System
```

The important point is that both implementations satisfy the same **editor-level responsibility**.

The File Manager does not change its purpose merely because the underlying operating-system API changes.

## Handles and File Descriptors

The Unix/POSIX model commonly represents open resources through file descriptors.

Windows has its own resource-handle model.

This difference illustrates why operating-system-specific types should not leak into the Core.

A poor design might expose platform resource types throughout the application:

```text
Editor State
    |
    +-- POSIX file descriptor
    |
    +-- Windows handle
```

This forces the Core to understand operating-system details.

A better design keeps such values inside the platform-facing implementation:

```text
+----------------------+
|     Editor Core      |
+----------+-----------+
           |
           | abstract operation
           v
+----------------------+
|     File Manager     |
+----------+-----------+
           |
     +-----+-----+
     |           |
     v           v
 POSIX       Windows
 descriptor   handle
```

The resource representation is therefore an implementation detail.

## Terminal Interaction Is the Larger Challenge

For this editor, filesystem portability is only part of the problem.

A terminal editor depends heavily on interactive terminal behavior.

The editor needs mechanisms for:

```text
Read keyboard input
Configure input behavior
Write terminal output
Obtain terminal dimensions
Control terminal state
Restore terminal state
```

The Unix implementation can use the POSIX terminal model.

A Windows implementation may need different console or terminal mechanisms.

The architectural boundary therefore becomes particularly valuable:

```text
                 Input Engine
                      |
                      v
                Input Handler
                      |
                      v
             Terminal Interface
                 /          \
                /            \
            POSIX          Windows
              |                |
        Unix terminal     Windows terminal
```

The Input Engine remains unchanged.

The Terminal Interface absorbs the environmental difference.

## ANSI Sequences Are a Separate Concern

Windows also illustrates why terminal protocols and operating-system APIs should not be confused.

The editor may generate terminal control sequences as part of rendering.

That is conceptually separate from the mechanism used to communicate with the terminal.

```text
Renderer
   |
   | terminal output
   v
Terminal Interface
   |
   | platform-specific output mechanism
   v
Terminal Environment
```

The Renderer should not need to know whether the underlying platform uses a Unix file descriptor or a Windows-specific handle.

Likewise, the Terminal Interface should not decide what the editor should render.

This preserves the responsibilities already established in the rendering architecture.

## Windows and the Renderer

The Renderer converts editor state into a terminal frame.

Its conceptual responsibility remains:

```text
Editor State
      |
      v
Prepare Frame
      |
      v
Document Area
      |
      v
Status Bar
      |
      v
Message Bar
      |
      v
Cursor
      |
      v
Terminal Output
```

That rendering model should not become Windows-specific.

The platform dependency begins when the generated output must actually be communicated to the terminal environment.

Therefore:

```text
Renderer
    |
    | frame/output
    v
Terminal Interface
    |
    +---- POSIX implementation
    |
    +---- Windows implementation
```

This preserves the existing separation between **rendering decisions** and **terminal interaction**.

## Windows and the File Manager

The same principle applies to persistence.

The editor's safe-save policy remains an editor requirement:

```text
Document
   |
   v
Encode text
   |
   v
Write temporary file
   |
   v
Replace destination
   |
   v
Clear Dirty Flag
```

The platform implementation provides the filesystem mechanisms necessary to perform those operations.

Therefore:

```text
Safe Save Policy
        |
        v
   File Manager
      /     \
     /       \
  POSIX    Windows
    |          |
 Unix FS    Windows FS
```

The safe-save algorithm should not be duplicated merely because the filesystem API differs.

Only the platform-specific mechanisms used to implement its individual operations may differ.

## Conditional Compilation Becomes More Important

Windows is one of the situations in which conditional compilation can become necessary.

For example, a platform-facing implementation might conceptually select an implementation according to the target platform:

```c
#if defined(_WIN32)

/* Windows implementation */

#else

/* POSIX implementation */

#endif
```

The important issue is **where this conditional code exists**.

This is acceptable:

```text
terminal.c
    |
    +-- Windows implementation
    |
    +-- POSIX implementation
```

It becomes problematic when the same platform checks appear throughout the application:

```text
document.c
cursor.c
command.c
search.c
undo.c
renderer.c
editor.c
...
```

The latter design turns platform selection into a global concern.

The former confines it to the component that actually needs different behavior.

Section 37.10 will examine conditional compilation in detail, so this section establishes only its architectural role.

## Windows as an Architectural Stress Test

Windows provides a stronger test of the platform boundary than another POSIX-oriented operating system.

Suppose the editor's Linux implementation is complete.

Adding Windows should ideally produce changes concentrated in:

```text
Platform Layer
    |
    +-- Terminal implementation
    +-- File implementation
    +-- Input implementation
    +-- Build configuration
    +-- Platform tests
```

The following should remain largely unchanged:

```text
Core
├── Document
├── Cursor
├── Viewport
├── Editor State
└── Undo/Redo

Application
├── Input Engine
└── Command Engine
```

If these components require extensive Windows-specific branches, the platform boundary has probably been placed too high in the architecture.

## Native Windows vs POSIX Compatibility Layers

Windows can also host environments that provide Unix-like or POSIX-oriented behavior.

This creates an important distinction.

```text
Windows
 |
 +---- Native Windows environment
 |
 +---- Unix-like compatibility environment
```

The native Windows implementation and a POSIX compatibility environment may expose different interfaces to the editor.

The editor should therefore distinguish between:

```text
Operating system
```

and:

```text
Execution environment / compatibility layer
```

This distinction becomes especially important when we discuss WSL in the next section.

For the architecture, the key principle remains:

> **Target the interfaces actually available to the running environment rather than assuming that the underlying machine determines the entire programming model.**

## Windows Support and Project Scope

The existence of Windows in the portability discussion does not mean that the first implementation must support Windows.

The project already establishes Linux/POSIX as the primary target.

Therefore, the appropriate engineering strategy is:

```text
Phase 1
Linux / POSIX
    |
    v
Validate architecture
    |
    v
Phase 2
Additional POSIX systems
    |
    v
macOS / BSD
    |
    v
Phase 3
Windows
```

This allows portability to influence the architecture from the beginning without forcing every platform to be implemented simultaneously.

That distinction is important for a textbook project: **designing for portability and implementing every platform are different tasks**.

## Summary

Windows demonstrates why portability must be based on explicit architectural boundaries.

The important principles are:

1. Windows does not share the same native POSIX environment as Linux, macOS, and BSD.
2. The editor's Core should nevertheless remain shared.
3. Native Windows APIs can be isolated behind platform-facing components.
4. Resource representations such as Windows handles should not leak into the Core.
5. Terminal interaction requires a platform-specific implementation boundary.
6. Rendering decisions should remain independent of the mechanism used to communicate with the terminal.
7. File-safety policies remain editor-level requirements even when filesystem mechanisms differ.
8. Conditional compilation should be localized to platform-sensitive components.
9. Windows provides a strong architectural stress test for the Platform Layer.
10. Designing for Windows does not require implementing Windows in the initial Linux-focused version.

The resulting model is:

```text
                    Editor Core
                        |
                        v
               Stable Interfaces
                        |
              +---------+---------+
              |                   |
          POSIX Platform     Windows Platform
              |                   |
       Linux/macOS/BSD        Windows APIs
```

The next section addresses **WSL**, which is particularly interesting because Windows can host a Linux environment. This forces us to distinguish the **host operating system**, the **execution environment**, and the **interfaces actually used by the editor**.
