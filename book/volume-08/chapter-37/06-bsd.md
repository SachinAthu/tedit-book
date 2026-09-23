# 37.6 BSD

BSD provides another useful portability reference because it reinforces a distinction already established in this chapter:

> **A system can share the POSIX model with Linux without being Linux.**

The BSD family includes operating systems with Unix-like interfaces and strong POSIX foundations. For our editor, BSD is therefore useful primarily as a test of whether the implementation has accidentally confused **POSIX behavior with Linux-specific behavior**.

This section does not attempt to survey individual BSD operating systems. The purpose is to understand what BSD contributes to the editor's portability model.

## BSD Within the Platform Model

The relationship can be represented as:

```text
                  Editor
                    |
                    v
           Platform Boundary
                    |
                    v
             POSIX Interfaces
                    |
          +---------+---------+
          |         |         |
        Linux     macOS      BSD
```

Linux, macOS, and BSD can share substantial portions of the platform-facing implementation because they expose common Unix/POSIX interfaces.

However, they remain distinct operating systems.

Therefore:

```text
POSIX compatibility
        ≠
identical operating system
```

This is the central lesson of the BSD target.

## Why BSD Is Useful for This Project

If Linux is the only Unix-like platform tested, it is easy to unintentionally introduce Linux-specific assumptions.

For example, a developer might use:

```text
Linux behavior
     ↓
assume it is Unix behavior
     ↓
assume it is portable
```

Testing on BSD provides another environment against which those assumptions can be challenged.

A healthier model is:

```text
POSIX requirement
       |
       +---- Linux
       |
       +---- macOS
       |
       +---- BSD
```

When the editor relies on a common interface, the same conceptual operation should be available across these environments.

When the implementation relies on something outside that common foundation, the dependency should be recognized as platform-specific.

## Common Interfaces

Many of the editor's fundamental operating-system interactions can be expressed through the same general Unix/POSIX model:

```text
File operations
Terminal I/O
File descriptors
Process environment
Terminal configuration
```

This allows the architecture to retain the same high-level structure:

```text
                 Editor Core
                      |
                      v
             Platform Interface
                      |
          +-----------+-----------+
          |           |           |
        Linux       macOS       BSD
```

The Core does not need to know which of these systems is underneath the interface.

## BSD Does Not Remove Platform Differences

The existence of common interfaces does not mean that every implementation detail can be ignored.

Differences may still exist in:

- system extensions,
- available APIs,
- default environments,
- terminal environments,
- filesystem behavior,
- compiler and toolchain environments,
- operating-system-specific facilities.

Therefore, portability should still follow the same rule:

> **Use common interfaces where they provide the required behavior, and isolate platform-specific facilities where they do not.**

The architecture should never depend on the assumption that "all Unix-like systems are identical."

## The Danger of Linux Extensions

One of the most important portability risks is accidentally depending on an extension provided by Linux rather than by the common interface being targeted.

Conceptually:

```text
             Application
                  |
                  v
             POSIX API
                  |
                  X
          Linux-specific API
                  |
                  v
                Linux
```

If a higher-level component directly depends on the Linux-specific interface, the dependency becomes difficult to remove later.

The better structure is:

```text
             Application
                  |
                  v
        Platform Interface
                  |
        +---------+---------+
        |                   |
      POSIX             Linux-specific
   implementation       implementation
        |                   |
        v                   v
   Linux/macOS/BSD        Linux
```

The second structure makes the dependency explicit and local.

## BSD and Terminal Interaction

BSD also reinforces the distinction between the operating system and the terminal emulator.

The editor's terminal path remains conceptually:

```text
Keyboard
    |
    v
Terminal Emulator
    |
    v
Terminal / PTY
    |
    v
Editor
```

The editor should not assume that because the underlying operating system is BSD, every terminal environment behaves in a particular way.

Terminal behavior remains a separate concern.

### BSD and File Management

The same separation applies to file operations.

At the editor level:

```text
Save Document
```

At the platform level:

```text
File Manager
     |
     v
POSIX filesystem operations
     |
     v
BSD filesystem
```

The File Manager owns the interaction with the filesystem.

The Document does not need to know whether the filesystem belongs to Linux, macOS, or BSD.

The save policy established earlier—such as writing through a temporary file and replacing the destination safely—remains an editor requirement. The exact system mechanisms used to realize that policy belong to the platform-facing implementation.

### BSD as an Architectural Test

BSD provides a useful test of the architecture for the same reason that macOS does.

Suppose a BSD implementation is introduced.

A healthy architecture should allow most of the following to remain unchanged:

```text
Document
Cursor
Viewport
Editor State
Command Engine
Search
Undo/Redo
Buffer management
Window management
```

Changes should primarily occur around:

```text
Platform Layer
Terminal interaction
File interaction
Build configuration
Platform-specific tests
```

If large portions of the Core require changes simply to compile or operate on BSD, that suggests platform assumptions have leaked upward.

Again, this is not an absolute rule. Genuine behavioral differences can require changes at an interface or implementation boundary.

The important question is:

> **Where do those changes occur?**

## Avoiding a "Unix Layer" That Is Too Broad

There is another architectural danger.

After recognizing that Linux, macOS, and BSD share many interfaces, it may be tempting to create a giant abstraction called:

```text
UnixPlatform
```

and put every Unix-specific concern into it.

That can become too broad.

For example:

```text
UnixPlatform
├── Terminal
├── Files
├── Input
├── Processes
├── Signals
├── Environment
├── Configuration
├── Rendering
└── Editor State
```

This recreates the coupling that the Platform Layer was intended to prevent.

The better approach is to keep abstractions aligned with **actual responsibilities**.

For this editor, terminal interaction and file interaction are distinct concerns even when both use POSIX interfaces.

```text
Platform
├── Terminal Interface
├── Input Handler
└── File Manager
```

The fact that multiple components use POSIX does not mean they must become one component.

## Shared Code Should Be Earned

Portability also requires discipline when deciding what code to share.

There are three reasonable possibilities:

```text
1. Common implementation
       |
       +---- works correctly everywhere

2. Common implementation
       |
       +---- small isolated platform differences

3. Separate implementations
       |
       +---- substantially different mechanisms
```

The choice should be based on actual behavior and interface requirements.

It should not be based on a desire to maximize the number of shared source files.

The goal is:

> **Share code where the abstraction is genuinely common; isolate code where platform behavior genuinely differs.**

This avoids both unnecessary duplication and overly complicated abstractions.

## BSD and the Editor's Core

The strongest portability test remains the Core Layer.

If the editor's Document implementation contains no operating-system dependencies, then it should not matter whether the program is running on:

```text
Linux
macOS
BSD
```

The Document still represents the same logical text.

Likewise, the Cursor still represents the same logical position.

The Viewport still represents the same visible region.

This gives us:

```text
              Platform
                 |
       +---------+---------+
       |                   |
    BSD APIs          Linux APIs
       |                   |
       +---------+---------+
                 |
                 v
          Stable Core Model
```

The platform changes the **mechanism of interaction with the environment**, not the meaning of the editor's fundamental data structures.

## What BSD Teaches Us

BSD strengthens several principles established in the previous sections.

First:

> POSIX is an important portability boundary.

Second:

> POSIX compatibility does not mean identical behavior.

Third:

> Linux-specific APIs should remain isolated when they are required.

Fourth:

> Platform-specific behavior should not leak into the Core.

Finally:

> Adding a Unix-like platform is an architectural test, not merely a compilation exercise.

A project that works on Linux only because its architecture accidentally depends on Linux is not genuinely portable.

A project that can use the same editor model with different platform implementations has a much stronger portability foundation.

## Summary

BSD provides a useful third perspective on the Unix/POSIX portability model.

The key principles are:

1. BSD shares important POSIX concepts with Linux and macOS.
2. POSIX compatibility does not guarantee identical system behavior.
3. BSD helps expose assumptions that are accidentally Linux-specific.
4. Common interfaces should be preferred where they satisfy the editor's requirements.
5. Platform-specific extensions should remain isolated.
6. The Platform Layer should remain organized around actual responsibilities rather than becoming one giant "Unix" abstraction.
7. The Core should remain unchanged by the choice between Linux, macOS, and BSD wherever the editor's semantics are identical.
8. Portability should be measured by the containment of platform-specific changes, not simply by whether the program compiles.

The progression so far is therefore:

```text
37.3 POSIX and Platform APIs
          |
          v
37.4 Linux
          |
          v
37.5 macOS
          |
          v
37.6 BSD
```

Linux establishes the reference implementation, macOS demonstrates portability across closely related systems, and BSD reinforces the need to distinguish **POSIX interfaces from Linux-specific assumptions**.

Next, we move outside the Unix/POSIX family to **Windows**, where the platform boundary becomes much more significant.
