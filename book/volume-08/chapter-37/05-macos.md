# 37.5 macOS

macOS is a useful second portability target because it demonstrates an important property of the architecture: **a platform can be different from Linux while still sharing the POSIX foundation used by the editor.**

The purpose of this section is not to teach macOS programming. The editor does not need a separate macOS-oriented architecture. Instead, macOS helps us identify which parts of the Linux implementation are genuinely portable and which parts are merely Linux-specific assumptions.

## macOS Within the Portability Model

The relationship can be represented as:

```text
                 Editor
                    |
                    v
           Platform Boundary
                    |
          +---------+---------+
          |                   |
       POSIX              macOS-specific
      interfaces             behavior
          |                   |
          +---------+---------+
                    |
                    v
                  macOS
```

Much of the editor's system interaction can therefore use the same general interfaces as Linux.

This is exactly why the platform boundary matters.

If the editor has been written directly against Linux-specific mechanisms, moving to macOS becomes a large rewrite.

If the editor has been written against appropriate common interfaces, much of the implementation can remain unchanged.

## What Can Remain Common?

The editor's higher-level behavior should not need to change merely because the operating system has changed.

The following concepts remain editor concepts:

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

Likewise, common system operations can often use the same POSIX-oriented approach:

```text
Open file
Read bytes
Write bytes
Close file
Rename file
Read terminal input
Write terminal output
```

Conceptually:

```text
+------------------+
|   Editor Core    |
+--------+---------+
         |
         v
+------------------+
| Common interface |
+--------+---------+
         |
     +---+---+
     |       |
     v       v
   Linux   macOS
```

The important point is that **source-code reuse should follow interface compatibility**, not merely the fact that both systems are Unix-like.

## macOS Is Not Linux

Shared interfaces do not imply identical platforms.

macOS and Linux have different operating-system implementations, system environments, and surrounding tooling.

Therefore, code should not make assumptions such as:

```text
POSIX-compatible
        =
Linux-compatible
        =
identical behavior
```

A more accurate model is:

```text
             Common interfaces
                    |
          +---------+---------+
          |                   |
       Linux                macOS
          |                   |
   implementation       implementation
```

The common interface provides a portability foundation.

It does not erase platform differences.

## Terminal Interaction

Terminal interaction is particularly important for the editor.

The high-level operation remains:

```text
Read terminal input
        |
        v
Convert input into editor events
        |
        v
Process event
```

The editor should not need separate versions of the Input Engine merely because the process is running on macOS.

Instead:

```text
                 Input Engine
                      |
                      v
                Input Handler
                      |
                      v
             Platform interface
                 /          \
                /            \
           Linux            macOS
```

The Input Handler and Terminal Interface contain the environmental details.

The Input Engine remains concerned with editor-level input and commands.

This preserves the same architectural separation established earlier.

## Terminal Modes

The editor's terminal-mode requirements were already established in the earlier chapters on terminal input and raw mode.

Chapter 37 does not need to re-explain how terminal modes work.

The portability question is different:

> **Can the same terminal abstraction be implemented on macOS without changing the editor's higher-level input architecture?**

For the POSIX-oriented design, the answer is generally yes.

The important architectural consequence is:

```text
Input Engine
     |
     | editor-level events
     v
Input Handler
     |
     | platform-specific terminal operations
     v
macOS terminal environment
```

The macOS implementation should satisfy the same interface expected by the higher layers.

## File Operations

The same principle applies to file management.

The editor's save operation is an editor-level operation:

```text
SAVE_DOCUMENT
```

The File Manager converts that operation into filesystem activity.

On macOS, the implementation can continue to use the common POSIX-oriented model where appropriate.

```text
Command Engine
      |
      v
   File Manager
      |
      v
POSIX filesystem interface
      |
      v
    macOS
```

The save algorithm itself should remain an editor policy rather than becoming a collection of Linux-specific assumptions.

This is especially important because Chapter 23 established atomic replacement and failure handling as editor-level file-safety requirements. The portability layer provides the mechanisms; it should not redefine the safety policy for each operating system.

## The Terminal Emulator Still Matters

Operating-system portability is only part of terminal portability.

Even on the same operating system, the editor can run inside different terminal environments.

For example:

```text
                    macOS
                      |
          +-----------+-----------+
          |                       |
      Terminal A              Terminal B
          |                       |
          +-----------+-----------+
                      |
                    Editor
```

The operating system and terminal emulator are separate components of the runtime environment.

Therefore, a successful macOS implementation does not mean that every terminal environment will behave identically.

This reinforces an important principle:

> **The Platform Layer must isolate operating-system dependencies, while the terminal-facing components must also avoid unnecessary assumptions about a particular terminal emulator.**

The detailed terminal-behavior implications will be addressed later in Section 37.9.

## Shared Implementation vs Separate Implementation

Not every platform-facing module necessarily needs a separate implementation.

There are several possibilities.

### Completely shared code

If an implementation relies only on common interfaces and behaves correctly on both platforms:

```text
terminal.c
     |
     +---- Linux
     |
     +---- macOS
```

the same implementation may be usable.

### Shared implementation with small platform differences

A module may contain a small amount of isolated conditional code:

```text
Common implementation
        |
        +---- Linux-specific detail
        |
        +---- macOS-specific detail
```

### Separate platform implementations

If the underlying mechanisms differ substantially:

```text
platform/
├── posix/
│   └── terminal.c
└── windows/
    └── terminal.c
```

the common interface can remain stable while implementations differ.

The architectural objective is therefore **not to force all platforms into one implementation**.

The objective is to prevent implementation differences from propagating into unrelated editor components.

## macOS as a Portability Test

macOS provides a useful architectural test.

Suppose the editor works on Linux and we attempt to support macOS.

If the changes are largely limited to:

```text
Platform Layer
     +
Build configuration
     +
Platform-specific testing
```

the architecture is behaving as intended.

If instead we discover changes required throughout:

```text
Document
Cursor
Command Engine
Undo/Redo
Search
Renderer
Editor State
...
```

then platform-specific assumptions have leaked into higher layers.

This gives us a practical measure of architectural quality:

> **The cost of adding a platform is an indicator of how effectively platform dependencies have been isolated.**

This is not an absolute metric—platforms can genuinely differ—but widespread changes in platform-independent code are a warning sign.

## What Should Not Be Assumed

The following assumptions should be avoided:

```text
"macOS is Unix-like, therefore everything behaves like Linux."

"POSIX means every terminal behaves identically."

"If it works on Linux, the same terminal behavior is guaranteed on macOS."

"Every system exposes the same extensions."

"Platform-specific behavior can be ignored because the APIs look similar."
```

A better approach is:

```text
Common requirement
       |
       v
Common interface where possible
       |
       v
Platform-specific implementation where necessary
       |
       v
Platform-specific validation
```

This is the same strategy used throughout the architecture: define the responsibility first, then isolate the mechanism used to fulfill it.

## macOS and the Project's Architecture

The existing architecture already gives us the necessary separation.

The Core Layer contains the editor's fundamental state and data structures, while platform-facing components handle interaction with the environment.

The project therefore does not need a separate:

```text
MacDocument
MacCursor
MacCommandEngine
```

There should instead be one editor model with platform-specific implementations only where the environment actually differs.

```text
                    Editor
                      |
          +-----------+-----------+
          |                       |
      Platform-independent    Platform-dependent
          |                       |
      Document                  Terminal
      Cursor                    File access
      Commands                  Input
      Viewport                  OS interaction
          |                       |
          +-----------+-----------+
                      |
                macOS environment
```

This distinction prevents portability from becoming duplication of the entire application.

## The Main Lesson

macOS demonstrates why portability should be based on **boundaries rather than assumptions**.

The editor does not need to know:

> "Am I running on Linux or macOS?"

for every operation.

Instead, the relevant platform-facing component should know how to perform its responsibility in the current environment.

The higher-level editor should continue to reason in terms of:

```text
Read Input
Execute Command
Save Document
Render Frame
Move Cursor
```

rather than:

```text
Call Linux API
Call macOS API
```

That distinction keeps the editor's conceptual model stable while allowing the implementation underneath it to vary.

## Summary

macOS demonstrates the practical value of the platform boundary.

The key principles are:

1. macOS shares important POSIX foundations with Linux.
2. Shared interfaces allow substantial implementation reuse.
3. POSIX compatibility does not imply identical behavior.
4. Terminal interaction remains platform-sensitive.
5. File operations can use common interfaces where appropriate.
6. Platform-specific implementation differences should remain near the Platform Layer.
7. The Core and Application layers should not be duplicated for each operating system.
8. The effort required to add macOS provides a useful test of whether platform dependencies have been properly isolated.

The goal is therefore not **Linux code versus macOS code**.

It is:

```text
                 Stable Editor
                      |
               Stable Interfaces
                      |
          +-----------+-----------+
          |                       |
       Linux                  macOS
   implementation         implementation
```

The next section examines **BSD**, which provides another useful perspective: it reinforces the distinction between a portable POSIX-oriented design and assumptions that are specifically tied to Linux.
