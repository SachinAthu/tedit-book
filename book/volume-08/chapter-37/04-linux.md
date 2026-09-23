# 37.4 Linux

Linux is the reference platform for this project. The editor is designed first against a Linux/POSIX environment, while macOS, BSD, Windows, and WSL are considered portability targets.

This distinction is important. The goal is not to make Linux behavior disappear behind an abstract layer. Linux is the **primary implementation environment** against which the architecture is developed and validated. Portability means that platform-specific assumptions remain isolated enough that other implementations can be introduced later. The SRS explicitly identifies Linux as the primary platform and POSIX-compatible systems as the target environment.

## Why Linux Is the Reference Platform

A project needs a concrete environment against which its behavior can be defined and tested.

For this editor, that environment is Linux.

```text
                 Editor Design
                      |
                      v
              +---------------+
              |     Linux     |
              | Reference OS  |
              +---------------+
                      |
          +-----------+-----------+
          |           |           |
        macOS        BSD       Windows
       portability  portability portability
```

Linux therefore provides the baseline for:

- terminal interaction,
- POSIX APIs,
- file operations,
- terminal mode management,
- process and descriptor behavior,
- development and testing.

This does **not** mean that Linux-specific assumptions should be scattered throughout the editor.

Instead:

```text
Linux-specific behavior
          |
          v
    Platform boundary
          |
          v
    Stable editor behavior
```

The reference platform establishes the initial implementation. The architecture determines how much of that implementation remains reusable elsewhere.

## Linux and POSIX

Most of the system-level facilities required by the editor can be expressed through interfaces commonly available on POSIX systems.

This is one reason Linux is a useful reference platform.

The relationship can be viewed as:

```text
+-----------------------------+
|       Editor Components     |
+-----------------------------+
              |
              v
+-----------------------------+
|       POSIX Interfaces      |
+-----------------------------+
              |
              v
+-----------------------------+
|       Linux Kernel / OS     |
+-----------------------------+
              |
              v
+-----------------------------+
|       Terminal / Files      |
+-----------------------------+
```

The application does not normally need to know the internal implementation of these facilities.

For example, the File Manager needs reliable file operations. It does not need the Document model to understand how Linux implements those operations.

Likewise, the Terminal Interface needs to configure and communicate with the terminal environment. The Cursor Engine should not need to know how Linux represents terminal devices.

This preserves the dependency direction established by the system architecture. The Core remains independent of platform mechanisms, while platform-facing components interact with them.

## File Descriptors

Linux exposes many resources through **file descriptors**.

A file descriptor is an integer identifying an open resource within a process.

For a terminal editor, descriptors are particularly relevant because terminal input and output are commonly accessed through standard streams or their underlying descriptors.

Conceptually:

```text
             Editor Process
                   |
        +----------+----------+
        |          |          |
       stdin     stdout     stderr
        |          |          |
        v          v          v
      input      output     diagnostics
```

The editor's normal runtime therefore has separate paths for:

- input,
- terminal output,
- diagnostic output.

This separation is important for debugging.

For example, the Renderer should produce terminal output through the normal terminal-output path, while diagnostics should not accidentally become part of the rendered screen.

The project's debugging architecture already treats diagnostics as a separate concern rather than mixing diagnostic information into normal editor rendering.

## The Terminal as a Device Interface

From the editor's perspective, the terminal is not simply a screen.

It is an interactive input/output environment.

```text
Keyboard
   |
   v
Terminal Emulator
   |
   v
PTY / Terminal Interface
   |
   +------> input ------> Editor
   |
   <------ output ------- Editor
```

The editor receives input bytes and sequences from the terminal environment and sends output bytes and escape sequences back.

This explains why terminal operation belongs in the Platform Layer.

The Core should reason about concepts such as:

```text
Arrow Up
Character 'a'
Backspace
Enter
Command
```

rather than:

```text
ESC
[
A
```

The conversion from terminal representation to editor-level events belongs to the input path already established by the architecture.

## Terminal Modes on Linux

Interactive terminal applications often need control over how terminal input is delivered.

For this editor, raw terminal operation was selected earlier as an architectural decision. Chapter 5 already explains raw mode and terminal state management; this section therefore does not repeat that mechanism.

The important portability observation is **where that mechanism belongs**.

```text
+----------------------+
|     Input Engine     |
|  editor-level input  |
+----------+-----------+
           |
           v
+----------------------+
|    Input Handler     |
+----------+-----------+
           |
           v
+----------------------+
|   Terminal Interface |
|                      |
| Linux/POSIX terminal |
|      operations      |
+----------------------+
```

The Input Engine should not directly manipulate Linux terminal configuration.

Instead, the Terminal Interface owns the platform interaction.

This follows the existing component responsibilities defined by the architecture: terminal operations belong to the Terminal Interface rather than the higher-level editor components.

## Terminal Dimensions

A full-screen editor must know the dimensions of the terminal.

Those dimensions affect:

- viewport size,
- status bar placement,
- message bar placement,
- rendering,
- cursor visibility,
- scrolling.

The architectural dependency is:

```text
Terminal Environment
        |
        | dimensions
        v
Terminal Interface
        |
        v
Editor State / Viewport
        |
        v
Renderer
```

The important principle is that the **mechanism used to obtain the dimensions is platform-facing**, while the dimensions themselves are editor data.

For example:

```text
Platform-specific mechanism
            |
            v
       rows, columns
            |
            v
     Editor-level data
```

Once the dimensions have been obtained, the Core does not need to know how they were discovered.

This is another example of the platform boundary introduced in Section 37.2.

## Linux Filesystem Interaction

The editor also depends on the operating system for persistent files.

The Document exists in memory:

```text
File
 |
 | load
 v
Document
 |
 | edit
 v
Modified Document
 |
 | save
 v
File
```

Linux provides the underlying filesystem operations, but the editor's persistence policy is higher-level.

The File Manager is responsible for turning the editor's save request into filesystem operations.

This distinction was established earlier when file safety and atomic replacement were discussed. The Linux-specific APIs are implementation mechanisms; the **safe-save policy is an editor requirement**.

Therefore:

```text
Safe Save Policy
       |
       v
   File Manager
       |
       v
 Linux/POSIX APIs
       |
       v
 Filesystem
```

The policy should not become tied to a particular Linux system call merely because Linux is the reference platform.

## Linux-Specific Code Should Have a Home

A common portability problem is allowing Linux-specific code to appear wherever it happens to be convenient.

For example:

```text
document.c
cursor.c
renderer.c
command.c
input.c
terminal.c
```

should not gradually become filled with Linux-specific branches.

Instead, platform-sensitive code should have an identifiable home.

The project structure already follows a module-oriented organization in which responsibilities are separated into modules.

Conceptually:

```text
src/
|
+-- core/
|   +-- document/
|   +-- cursor/
|   +-- viewport/
|   +-- editor_state/
|
+-- input/
|
+-- command/
|
+-- render/
|
+-- terminal/
|
+-- file/
|
+-- platform/
```

The exact directory organization is defined by the project's project-structure decisions; the important principle here is that platform-sensitive responsibilities remain identifiable.

## Linux as a Baseline, Not a Shortcut

There is a subtle danger in having a primary platform.

Because Linux is the development target, it is tempting to write code according to whatever Linux behavior happens to work.

For example:

```text
"It works on my Linux terminal."
```

is not sufficient evidence of portability.

The stronger question is:

> **Is this behavior part of the editor's contract, part of POSIX, or merely an observed property of this particular environment?**

These are different things.

```text
Editor contract
      |
      +---- must preserve

POSIX behavior
      |
      +---- preferred common interface

Linux behavior
      |
      +---- isolate when platform-specific

Terminal emulator behavior
      |
      +---- validate rather than assume
```

This distinction prevents accidental dependencies from becoming architectural assumptions.

## Linux-Specific Assumptions

Some assumptions are reasonable for the reference implementation.

For example:

- the primary development environment is Linux;
- POSIX interfaces are available;
- terminal operation follows the Unix/POSIX model;
- the editor can use the platform facilities defined by the project architecture.

These assumptions are already reflected in the SRS and project architecture.

However, assumptions should remain **explicit**.

An assumption that is hidden inside a Core component is difficult to replace.

An assumption isolated inside the Platform Layer is much easier to replace.

That gives us a useful rule:

> **The problem is not having platform assumptions. The problem is allowing platform assumptions to become invisible dependencies throughout the system.**

## What Should Remain Platform-Independent?

Even when running exclusively on Linux, the following concepts should remain independent of Linux:

```text
Document
Cursor
Viewport
Editor State
Commands
Undo/Redo
Search
Configuration
Buffer management
Window management
Rendering decisions
```

The following are naturally platform-facing:

```text
Terminal mode configuration
Terminal dimensions
Terminal input acquisition
Terminal output
File descriptor operations
Filesystem access
Platform-specific diagnostics
```

The boundary can therefore be summarized as:

```text
+--------------------------------------+
|          Platform Independent        |
|                                      |
| Document  Cursor  Viewport           |
| Editor State  Commands  Search       |
| Undo/Redo  Buffers  Windows          |
+------------------+-------------------+
                   |
                   | interfaces
                   v
+--------------------------------------+
|             Platform                 |
|                                      |
| Terminal  Input  Files  OS services  |
+--------------------------------------+
                   |
                   v
+--------------------------------------+
|        Linux / POSIX Environment     |
+--------------------------------------+
```

This is the same architectural separation that allows the editor to remain modular while interacting with its environment.

## Summary

Linux is the reference platform for the editor, but it should not become an implicit dependency of the entire system.

The important principles are:

1. **Linux provides the primary development and execution environment.**
2. **POSIX provides the principal common system interface.**
3. **Terminal interaction belongs to the Platform Layer.**
4. **File descriptors and filesystem operations are platform-facing mechanisms.**
5. **Terminal dimensions and operating-system resources should be converted into platform-independent editor data.**
6. **Linux-specific assumptions should remain explicit and localized.**
7. **The Core should remain independent of Linux even though Linux is the reference platform.**
8. **Linux behavior must not automatically be treated as portable behavior.**

The architecture therefore does not attempt to eliminate Linux dependencies. It **contains them**.

The next section examines **macOS**. The focus will be on what remains identical because of the POSIX foundation and what must still be treated as a platform-specific behavior rather than assumed to be identical to Linux.
