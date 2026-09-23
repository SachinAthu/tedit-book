# 37.8 WSL

Windows Subsystem for Linux (WSL) introduces an important distinction into our portability model.

A computer may be running Windows as its host operating system while the editor itself executes inside a Linux environment with Linux/POSIX interfaces.

Therefore, the statement:

> "The editor is running on Windows."

is not sufficient to determine which platform APIs the editor should use.

For portability, we must distinguish between the **host system** and the **execution environment**.

## The Host Is Not Necessarily the Target Environment

Consider this simplified model:

```text
Physical Computer
        |
        v
     Windows
        |
        v
       WSL
        |
        v
   Linux environment
        |
        v
     Editor
```

The machine is physically running Windows, but the editor may see a Linux/POSIX-oriented environment.

This means the editor's platform boundary should be determined by the **interfaces available to the running program**, not simply by the operating system installed on the physical machine.

This follows directly from the project's existing requirement that the editor target POSIX-compatible systems, with Linux as the primary reference platform.

## WSL Can Use the Linux Implementation

From the editor's perspective, a WSL execution environment can often fit naturally into the Linux/POSIX side of the architecture:

```text
                 Editor
                    |
                    v
             Platform Layer
                    |
                    v
              POSIX APIs
                    |
                    v
             Linux environment
                    |
                    v
                   WSL
                    |
                    v
                Windows
```

This is fundamentally different from compiling the editor as a native Windows application.

The distinction can be represented as:

```text
Native Windows:

Editor
  |
  v
Windows APIs
  |
  v
Windows


WSL:

Editor
  |
  v
POSIX / Linux APIs
  |
  v
Linux environment
  |
  v
WSL
  |
  v
Windows
```

Therefore, the same editor source can potentially operate in WSL without introducing a separate Windows implementation.

## WSL Does Not Mean "Windows Support"

This distinction is important for the scope of the project.

Supporting the editor under WSL does **not** automatically mean that the editor supports native Windows.

These are different execution environments:

```text
                Windows machine
                     |
          +----------+----------+
          |                     |
          v                     v
     Native Windows            WSL
          |                     |
    Windows APIs          Linux/POSIX APIs
          |                     |
          v                     v
   Windows implementation   Linux implementation
```

A successful Linux build running under WSL demonstrates compatibility with the Linux environment provided by WSL.

It does not prove that the editor can be compiled as a native Windows terminal application.

This distinction prevents us from incorrectly treating WSL as a substitute for Windows portability.

## Why WSL Is Relevant to Our Architecture

WSL provides an interesting validation case for the platform boundary.

Suppose the editor is implemented against POSIX interfaces:

```text
Editor
   |
   v
POSIX interface
   |
   v
Linux
```

If the same implementation works in WSL:

```text
Editor
   |
   v
POSIX interface
   |
   v
Linux environment
   |
   v
WSL
```

then the editor has demonstrated that it does not depend unnecessarily on a particular physical machine or host environment.

The application is interacting with the **provided programming environment**, not directly with the host hardware.

## Terminal Architecture Still Matters

WSL does not eliminate terminal considerations.

The editor still follows the same conceptual pipeline:

```text
Keyboard
   |
   v
Terminal Environment
   |
   v
Input Handler
   |
   v
Input Parser
   |
   v
Input Engine
```

The existing architecture explicitly separates raw terminal input, parsing, and editor-level input processing.

Therefore, WSL does not require the Input Engine to understand WSL.

If terminal-specific behavior differs, that difference belongs closer to:

```text
Terminal Interface
        +
Input Handler
        +
Input Parser
```

rather than inside the Core.

## WSL and Filesystem Boundaries

WSL also illustrates that the environment in which the editor executes can have access to different filesystem domains.

Conceptually:

```text
             Editor
                |
                v
        Linux filesystem
                |
        +-------+-------+
        |               |
    Linux files     Windows files
                   through WSL
```

The important point for the editor is not to make assumptions about the underlying physical storage mechanism.

The File Manager should operate through the filesystem interface available to its execution environment.

Its responsibility remains:

```text
Load file
   ↓
Populate Document

Save file
   ↓
Write Document
   ↓
Report success/failure
```

The existing architecture deliberately isolates filesystem APIs inside the File Manager.

That design is particularly useful in environments such as WSL.

### Host Detection Is Usually the Wrong Abstraction

A poorly designed application might try to determine:

```text
Am I running on Windows?
```

and then select its implementation based on that answer.

That can be misleading in WSL.

The more useful question is:

> **Which system interfaces are available to this process?**

For example:

```text
What terminal interface is available?
What filesystem interface is available?
What compiler target was selected?
What operating-system APIs can this process use?
```

This leads to a more robust architecture:

```text
Build / Runtime Environment
            |
            v
Available Platform Interface
            |
            v
Platform Implementation
```

rather than:

```text
Physical Host
      |
      v
Guess platform
      |
      v
Choose implementation
```

## Build Configuration Is Part of Portability

WSL also demonstrates that portability is not limited to source code.

A project must also be buildable in the target environment.

For example:

```text
Source Code
    |
    v
Compiler
    |
    v
Build Configuration
    |
    v
Executable
```

A Linux-oriented build running under WSL may use the Linux compiler and libraries available within that environment.

A native Windows build requires a different toolchain and potentially different platform libraries.

Therefore:

> **A platform implementation is not complete until its source code, dependencies, build configuration, and runtime environment are all compatible.**

This does not introduce a new build-system architecture into the book. It establishes the portability implication that build configuration is part of the platform boundary.

## WSL and the Reference Platform

WSL does not change the project's reference platform.

The SRS explicitly defines Linux as the primary development and reference platform.

Therefore:

```text
Primary platform:
Linux / POSIX

Additional execution environment:
WSL
```

The architecture should not be modified simply to make WSL a special target.

If the Linux implementation works correctly within WSL, WSL benefits from the existing Linux/POSIX implementation.

This is preferable to introducing unnecessary code such as:

```text
Linux
WSL
macOS
BSD
Windows
```

when WSL does not actually require a separate platform implementation.

## WSL as Evidence of Good Isolation

WSL provides a useful architectural test similar to the macOS and BSD tests discussed earlier.

If the editor runs under WSL with little or no modification:

```text
Linux implementation
        |
        v
      WSL
        |
        v
     Works
```

that suggests the implementation depends on the Linux/POSIX environment rather than on assumptions about a particular physical Linux installation.

Conversely, if the editor fails because it assumes something specific about the host environment, that reveals an unnecessary dependency.

The distinction is:

```text
Portable assumption:
"POSIX terminal interface is available."

Fragile assumption:
"I am physically running on a Linux computer."
```

The first is an interface-level assumption.

The second is a host-level assumption.

Portability should favor the former.

## WSL Does Not Eliminate Terminal Variations

One further distinction is important.

Even if the editor sees a Linux/POSIX environment, its terminal may be provided through a Windows-hosted terminal application.

Therefore:

```text
Windows
   |
   v
WSL
   |
   v
Linux process
   |
   v
Terminal environment
```

contains several layers.

The editor should continue to rely on the terminal abstractions already established by the architecture rather than assuming that "Linux process" implies a particular terminal emulator.

This connects directly to the later discussion of platform-specific terminal behavior.

## The Architectural Lesson

WSL demonstrates that **platform is a property of the execution environment and interfaces, not simply the host machine**.

For the editor:

```text
                 Host
                  |
          +-------+-------+
          |               |
      Windows           Windows
          |               |
   Native execution      WSL
          |               |
   Windows APIs       Linux/POSIX APIs
          |               |
   Windows platform   Linux platform
```

The editor should select its implementation according to the environment in which it actually executes.

This makes the platform boundary more precise.

## Summary

WSL is important because it prevents us from treating "Windows" as a single execution environment.

The key principles are:

1. A Windows host can provide a Linux/POSIX execution environment through WSL.
2. An editor running under WSL can use the Linux/POSIX implementation.
3. WSL support is not equivalent to native Windows support.
4. Platform selection should follow the interfaces available to the running process rather than the physical host alone.
5. Terminal and filesystem behavior should remain behind the existing Platform Layer.
6. Build configuration is also part of portability.
7. WSL should not require a separate editor architecture when the existing Linux implementation already satisfies its environment.
8. WSL provides a useful test of whether the editor depends on interfaces or on accidental host assumptions.

The overall platform model is now:

```text
                         Editor
                            |
                     Platform Boundary
                            |
          +-----------------+-----------------+
          |                 |                 |
       POSIX             POSIX            Windows
          |                 |                 |
       Linux             macOS/BSD       Native APIs
          |
         WSL
          |
       Windows host
```

The next section turns from operating-system identity to a more subtle problem: **platform-specific terminal behavior**. This is where Linux, macOS, BSD, Windows, WSL, and even different terminal emulators can expose differences that the editor must handle without contaminating its core architecture.
