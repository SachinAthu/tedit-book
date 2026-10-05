# 43.3 Nano — Simplicity as an Architectural Choice

GNU nano is a useful first case study because it provides a strong contrast with the increasingly sophisticated architectures we will examine later in this chapter.

Nano is a terminal text editor designed around a relatively direct interaction model. Its architecture does not attempt to turn the editor into a general-purpose extensible platform. Instead, it focuses on providing the fundamental operations required to edit text effectively in a terminal.

That makes Nano particularly relevant to our own project.

Our editor is also intentionally designed as a **small terminal editor**, with simplicity, clarity, and educational value treated as architectural priorities. The project requirements explicitly prioritize a lightweight, understandable architecture rather than feature completeness. 02-software-requirements

The comparison therefore gives us an opportunity to ask:

> **How far can a text editor go without introducing the architectural complexity of a large extensible editor?**

## A Direct Interaction Model

At a conceptual level, a simple terminal editor can be understood as a relatively direct pipeline:

```text
Keyboard
    |
    v
Input Processing
    |
    v
Editing Command
    |
    v
Document State
    |
    v
Terminal Rendering
```

There are still important boundaries inside this pipeline.

Input must be interpreted correctly. Commands must modify the appropriate state. The document must maintain its invariants. The renderer must transform the resulting state into a visible terminal frame.

But the overall system does not require a large hierarchy of independently extensible services.

This is an important distinction.

**Simple architecture does not mean absence of architecture.**

It means that the architecture contains only the boundaries justified by the system's requirements.

## Simplicity Is Not the Same as Poor Design

It is tempting to assume that a more sophisticated architecture is automatically a better architecture.

That is incorrect.

Suppose an editor has only these requirements:

- open a text file;
- display its contents;
- move the cursor;
- insert and delete text;
- save the file;
- provide basic terminal interaction.

Introducing a plugin runtime, extension host, remote procedure interface, multiple rendering processes, or a sophisticated asynchronous service architecture would increase the number of concepts the developer must understand.

Those mechanisms may be valuable in a much larger editor.

They are not automatically valuable here.

Our own architecture follows this principle. The project deliberately separates responsibilities such as input handling, command execution, document management, and rendering, but it avoids introducing mechanisms whose benefits are outside the baseline requirements.

This is consistent with the project's architectural philosophy: **simplicity is itself a design objective**.

## The Cost of Simplicity

Simplicity also has consequences.

A small editor may have fewer abstractions, but this can mean that some capabilities are harder to add later.

For example:

```text
Simple Editor
     |
     +-- Simple Commands
     +-- Simple State
     +-- Simple Rendering
     |
     v
Easy to understand
```

But extending it significantly may eventually produce:

```text
Simple Editor
     |
     +-- Plugins
     +-- Language Services
     +-- Multiple Windows
     +-- Background Tasks
     +-- Async Operations
     +-- Extension APIs
     |
     v
More architectural pressure
```

At some point, the original boundaries may no longer be sufficient.

This is one reason why mature editors often look very different from small educational editors.

They have accumulated requirements over time.

## Nano and Our Editor

The comparison with Nano helps reinforce several decisions in our own design.

Our editor is intentionally built around a small set of core responsibilities:

```text
                  Editor State
                       |
       +---------------+---------------+
       |               |               |
     Input          Commands        Renderer
       |               |               |
       v               v               v
   Input Model      Document       Screen Buffer
                       |
                       v
                  File Manager
```

The objective is not to minimize the number of source files or functions.

The objective is to keep **responsibility boundaries understandable**.

For example, our Input Parser should not become responsible for modifying the Document. Our Renderer should not become responsible for editing text. The File Manager should not become responsible for cursor movement.

These boundaries allow the implementation to remain simple without becoming unstructured.

## What Nano Teaches Us

The important lesson from this case study is therefore not a particular implementation technique.

It is an architectural principle:

> **Do not introduce complexity merely because more sophisticated editors use it.**

A small editor can be professionally designed while remaining small.

The quality of an architecture is determined by how well it satisfies its requirements, not by how many components it contains.

This provides an important baseline for the editors we examine next.

Vim introduces a substantially different interaction model. Its architecture demonstrates what happens when **commands, modes, motions, and composable editing operations** become central to the user experience.
