# 38.4 Installation

Building the editor produces an executable. **Installation is the process of placing that executable and its associated resources into a layout from which the software can be used as an installed application.**

This distinction is important.

A successful build does not automatically mean that the editor is ready for users.

```text
Source Code
    |
    v
Build
    |
    v
Executable
    |
    v
Installation
    |
    v
Installed Editor
```

The build answers:

> Can we produce the software?

Installation answers:

> Where should the produced software and its supporting files go?

## Build Output vs Installed Software

During development, the executable may exist inside a build directory:

```text
project/
├── src/
├── include/
├── tests/
└── build/
    └── debug/
        └── editor
```

That location is useful for development, but it is not necessarily an appropriate layout for an installed program.

An installed editor might instead look conceptually like:

```text
Installation Prefix/
├── bin/
│   └── editor
│
├── share/
│   └── editor/
│       └── documentation/
│
└── ...
```

The exact filesystem layout varies between operating systems.

The important distinction is:

```text
Build Directory
    → temporary/generated build environment

Installation Directory
    → user-facing software layout
```

The two should not be confused.

## The Installation Prefix

An installation normally has a **prefix**: the root location under which the editor's installed files are placed.

Conceptually:

```text
Prefix
  |
  +-- bin/
  |
  +-- share/
  |
  +-- other platform-specific locations
```

For example, a system installation might use a system-wide prefix, while a user might install the editor beneath a directory in their home environment.

The important property is that the installation layout should be determined by the installation system rather than hard-coded throughout the application.

This matters for portability.

Different platforms have different conventions for where executables, documentation, configuration, and other resources belong.

## Installing the Executable

The most important installed component is the editor executable itself.

Conceptually:

```text
Build
  |
  v
editor
  |
  | install
  v
<installation-prefix>/bin/editor
```

Once installed, the executable should be accessible through the platform's normal mechanisms.

On systems where the installation directory is included in the user's command search path, the user can invoke:

```text
editor
```

without needing to know where the executable physically resides.

This is one of the differences between **having an executable** and **having an installed application**.

## Installed Resources

The executable may not be the only thing that belongs to an installation.

Depending on the final editor design, an installation may also contain resources such as:

- documentation,
- example configuration files,
- default data,
- license information,
- other read-only resources.

However, not every file in the source repository should be installed.

For example, these are normally development artifacts:

```text
tests/
build/
CMake cache files
compiler-generated files
development scripts
```

An installation should contain only what the user or the installed software actually needs.

A useful distinction is:

```text
Source Repository
        |
        +-- Source files
        +-- Tests
        +-- Build configuration
        +-- Documentation source
        +-- Development files
        |
        v
     Build / Install Selection
        |
        v
Installed Software
        |
        +-- Executable
        +-- Required resources
        +-- User-facing documentation
```

## Installation Is Not Configuration

Installation and configuration are related, but they are different concerns.

Installation determines **where software components are placed**.

Configuration determines **how the installed editor behaves**.

For example:

```text
Installation
    |
    +-- editor executable
    +-- documentation
    +-- shared resources


User Configuration
    |
    +-- key bindings
    +-- tab width
    +-- other editor preferences
```

Chapter 33 already established the editor's configuration model:

```text
Built-in Defaults
       ↓
User Configuration
       ↓
Optional Command-Line Overrides
```

That configuration model should not be duplicated here.

Chapter 38 is concerned with getting the software onto the system; Chapter 33 is concerned with how the installed editor obtains its behavior settings.

## System Installation vs User Installation

An editor can conceptually be installed at different scopes.

A **system installation** makes the software available to multiple users on a machine.

A **user installation** places the software within a user's own environment.

The difference is primarily about ownership, permissions, and installation location.

```text
System Installation
    |
    v
System-wide location
    |
    v
Multiple users


User Installation
    |
    v
User-controlled location
    |
    v
Individual user
```

The book does not need to prescribe one universal installation location.

Instead, the build and installation system should support the conventions of the target platform.

This fits the portability strategy established in Chapter 37: platform-specific behavior should be contained at appropriate boundaries rather than scattered throughout the editor.

## CMake Installation Rules

CMake can describe which targets and resources should be installed.

This provides an important separation:

```text
CMake Build Rules
    |
    +-- What gets compiled?
    |
    v
Build Output


CMake Install Rules
    |
    +-- What gets installed?
    +-- Where does it go?
    |
    v
Installed Software
```

The source project can therefore define an installation model without requiring developers or users to manually copy files.

This is especially valuable for release preparation because the same installation procedure can be repeated consistently.

## Installation Should Be Testable

An installation procedure should itself be tested.

It is possible for the following sequence to occur:

```text
Build succeeds
      |
      v
Installation succeeds
      |
      v
Installed editor fails
```

For example, an installed executable might expect a resource that was accidentally omitted from the installation.

Therefore, installation should be treated as part of the software delivery process rather than as an administrative afterthought.

A useful verification sequence is:

```text
Clean Build
    |
    v
Install
    |
    v
Run Installed Editor
    |
    v
Verify Core Behavior
```

The test should exercise the **installed copy**, not the executable sitting in the build directory.

Otherwise, the test may accidentally verify the wrong environment.

## Clean Installation Testing

A particularly useful test is to install into a clean prefix.

Conceptually:

```text
Temporary Installation Prefix
            |
            v
        Install
            |
            v
      Installed Editor
            |
            v
          Test
```

This prevents the test from succeeding because files from an older installation happen to remain available.

A clean installation therefore provides stronger evidence that the installation rules are complete.

The distinction is similar to a clean build:

```text
Clean Build
    → verifies the build does not depend
      on stale generated files

Clean Installation
    → verifies the installation does not depend
      on stale installed files
```

## Installation and Portability

Installation is also affected by the platform on which the editor runs.

The project has deliberately chosen Linux as the primary reference platform while treating POSIX-compatible systems and Windows as portability targets. ADR-030 also establishes that platform-specific behavior should be isolated at the platform boundary.

The same principle applies to installation.

We should not assume that every platform uses the same installation layout.

Instead:

```text
Common Editor
      |
      v
Common Installation Intent
      |
      +----------+----------+
      |                     |
      v                     v
Platform A              Platform B
      |                     |
      v                     v
Platform-specific      Platform-specific
installation layout    installation layout
```

The application itself should not need to know the absolute installation prefix merely to function.

Where the editor genuinely needs to locate installed resources, that relationship should be designed deliberately rather than relying on accidental development-directory paths.

## What Should Not Be Installed

A release installation should not simply copy the entire project directory.

For example:

```text
Do not install:

.git/
build/
tests/
CMakeCache.txt
compiler-generated files
temporary files
development-only diagnostics
```

The source repository exists for developers.

The installation exists for users.

These are different artifacts with different purposes.

```text
Source Repository
        |
        | contains everything needed
        | to develop the project
        v
Build System
        |
        | selects what belongs
        | in the installation
        v
Installed Software
        |
        | contains only what is
        | required for use
        v
User
```

## Installation as a Release Boundary

Installation creates an important boundary in the release pipeline.

Before installation, we are primarily dealing with **project and build artifacts**.

After installation, we are dealing with **software as experienced by a user**.

```text
Development World
       |
       v
Source
       |
       v
Build
       |
       v
Release Executable
       |
       v
Installation
       |
       v
User Environment
```

This makes installation an important validation point.

If the editor works only from its build directory but fails after installation, the release process is incomplete even though compilation succeeded.

## Summary

**Building produces the editor; installation makes the editor available as software.**

The installation process should:

- place the executable in the appropriate location,
- install only required user-facing resources,
- respect platform conventions,
- avoid dependence on the source or build tree,
- support clean installation testing,
- and remain reproducible through the build system.

The distinction can now be summarized:

```text
Build
    → Produce the executable.

Installation
    → Place the executable and required resources
      into a usable system layout.
```

With installation defined, the next question is:

> **What exactly do we package and hand to users as a release?**

That leads to **Release Artifacts**.
