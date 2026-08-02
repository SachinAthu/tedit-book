# 7.6 Non-Functional Requirements

In the previous section, we identified the **functional requirements** of our editor. These requirements describe **what** the software must do.

However, functionality alone does not determine whether software is good.

Consider two text editors that provide exactly the same features:

- Both can open files.
- Both can save files.
- Both allow text editing.
- Both support cursor movement.

On paper, they satisfy the same functional requirements.

But imagine that one editor takes five seconds to respond to every key press, occasionally loses data while saving, and crashes whenever it encounters an unexpected error.

Although it provides all the required features, very few people would consider it a good editor.

This illustrates an important principle:

> **Software is judged not only by what it does, but also by how well it does it.**

These quality attributes are described by **non-functional requirements**.

## What Are Non-Functional Requirements?

A **non-functional requirement (NFR)** specifies the qualities or characteristics that the software should possess rather than the features it provides.

Instead of asking:

> **"What should the system do?"**

Non-functional requirements ask questions such as:

- How fast should it respond?
- How reliable should it be?
- How easy should it be to maintain?
- How portable should it be?
- How much memory should it use?

These requirements define the overall quality of the software.

Unlike functional requirements, which describe individual behaviors, non-functional requirements often influence the design of the entire system.

## Functional vs. Non-Functional Requirements

The difference can be summarized as follows:

```text
Functional Requirement

"What does the system do?"

↓

Open files
Save files
Move cursor
Insert text

----------------------------

Non-Functional Requirement

"How well does it do it?"

↓

Fast
Reliable
Maintainable
Portable
Responsive
```

Both categories are equally important.

A feature-rich application that performs poorly or is difficult to maintain is unlikely to be successful.

## Why Non-Functional Requirements Matter

Non-functional requirements often drive major architectural decisions.

Suppose we establish the following requirement:

> The editor shall respond immediately to every key press.

Although this sounds simple, it influences many later design choices.

For example:

- The renderer should redraw the screen efficiently.
- The editor should avoid unnecessary disk access during editing.
- The document should be stored in memory rather than modified directly on disk.
- The application should minimize unnecessary computation during the main event loop.

A single quality requirement can therefore affect multiple subsystems.

Unlike functional requirements, which often correspond to individual features, non-functional requirements shape the overall architecture of the software.

## Categories of Non-Functional Requirements

For our editor, we will focus on five major categories.

```text
Non-Functional Requirements

├── Performance
├── Reliability
├── Usability
├── Maintainability
└── Portability
```

These categories align well with the educational goals of this book, emphasizing not only how to build software, but also how to build it well. The complete SRS at the end of this chapter also formalizes **Extensibility** and **Security & Data Integrity** as additional NFR categories.

## Performance

Performance describes how efficiently the editor performs its work.

Examples include:

- The editor shall respond promptly to keyboard input.
- Screen updates shall occur without noticeable delay during normal editing.
- Memory usage shall remain reasonable for typical text files.

Performance does not necessarily mean making everything as fast as possible. Instead, it means delivering a responsive experience while keeping the implementation simple and understandable.

## Reliability

Reliability concerns the editor's ability to operate correctly under normal conditions.

Examples include:

- The editor shall not intentionally corrupt document contents.
- File operation failures shall be reported clearly to the user.
- Unexpected conditions shall be handled gracefully whenever possible.

Reliability becomes especially important when working with users' documents. Losing data is often far more serious than running slightly slower.

## Usability

Usability describes how easy the editor is to learn and operate.

Examples include:

- Keyboard shortcuts shall behave consistently.
- Status messages shall clearly communicate the current state of the editor.
- Cursor movement shall behave predictably.

Since our editor is terminal-based, users rely heavily on keyboard interaction and textual feedback. Clear and consistent behavior makes the editor much easier to use.

## Maintainability

Maintainability describes how easily the software can be understood, modified, and extended.

For this project, maintainability is particularly important because the editor serves as a teaching tool.

Examples include:

- The software shall use a modular architecture.
- Each component shall have a clearly defined responsibility.
- Functions should remain focused on a single task.
- Interfaces between modules should remain simple and well defined.

These goals directly support the architecture-first philosophy established for this book.

## Portability

Although this book focuses primarily on Linux, the overall design should avoid unnecessary platform dependencies whenever practical.

Examples include:

- The editor shall target POSIX-compliant systems.
- Platform-specific behavior shall be isolated where possible.
- The implementation shall rely primarily on standard C and POSIX APIs.

This approach makes it easier to adapt the editor to macOS and other POSIX-based systems in the future.

## Balancing Non-Functional Requirements

Non-functional requirements often compete with one another.

For example:

- Maximum performance may require more complex algorithms.
- Simpler code is generally easier to understand and maintain.
- Highly optimized implementations can become difficult to modify.

Throughout this book, we intentionally favor **clarity and educational value** over aggressive optimization.

For example, our renderer will initially redraw the entire screen rather than attempting incremental updates. While this is not always the most efficient strategy, it greatly simplifies the implementation and makes the rendering pipeline easier to understand. This design choice is documented as **ADR-002: Rendering Strategy**, where full redraw is chosen because of its simplicity.

Similarly, our initial document representation will prioritize readability over maximum editing performance, allowing readers to understand the core concepts before exploring more advanced data structures in later volumes.

## Engineering Insight

Functional requirements determine whether software is **capable**. Non-functional requirements determine whether that software is **practical**. Many professional engineering decisions—such as choosing an algorithm, dividing responsibilities between modules, or simplifying an implementation—are driven primarily by non-functional requirements rather than by new features.

## Summary

Non-functional requirements define the quality attributes of our editor. They describe how the software should perform, how reliable it should be, how easy it should be to use and maintain, and how portable it should remain across supported platforms. These requirements influence nearly every architectural decision that follows and provide the standards by which the overall quality of the editor can be evaluated.

With both the functional and non-functional requirements established, we can now examine the **constraints** under which our editor must operate and how those limitations shape the design of the system.
