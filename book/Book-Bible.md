# Building a Terminal Text Editor

## Book Bible (Project Constitution)

**Version:** 2.0  
**Status:** Active  
**Applies to:** Every volume, chapter, diagram, example, and code sample.

---

# 1. Purpose

This document defines the writing standards, educational philosophy, terminology, formatting rules, architectural decisions, and project conventions for the book.

Its purpose is to ensure the entire book feels as though it was written by a single author with a consistent teaching style and technical depth.

This document defines the global rules of the book. Where a rule is explicitly scoped to a particular volume, that scope takes precedence over a general rule. In particular, Volume X has an explicit implementation workflow defined below and in the Implementation Project Context.

---

# 2. Book Identity

## Title

**Building a Terminal Text Editor**

### Subtitle

**From First Principles to Professional Architecture**

---

# 3. Primary Goal

The goal of this book is **not** simply to build a text editor.

The real objective is to teach readers:

- Systems Programming
- Terminal Programming
- Software Architecture
- Data Structures
- Operating System Concepts
- Rendering Pipelines
- File Systems
- Engineering Trade-offs

The text editor is the vehicle used to teach these topics.

Readers should finish the book understanding _why_ software is designed the way it is—not just how to reproduce it.

---

# 4. Target Audience

Readers are expected to have:

- Basic C programming knowledge
- Functions
- Loops
- Arrays
- Structures
- Pointers
- Basic file I/O

No prior knowledge is assumed regarding:

- POSIX
- termios
- ANSI escape sequences
- TTY/PTY
- Text editor architecture
- Systems programming

---

# 5. Educational Philosophy

This book follows a strict philosophy:

## Explain before implementing — for Volumes I–IX.

Volumes I–IX establish the concepts, architecture, algorithms, engineering practices, and design decisions required to build the editor.

Every concept should answer:

1. What problem exists?
2. Why does it exist?
3. What solutions exist?
4. Why was this solution chosen?
5. What are the trade-offs?

This explain-before-implementing rule governs the theory volumes. Volume X is the deliberate implementation phase and follows its own code-first construction workflow defined in Section 6.

---

## Build understanding, not memorization.

Readers should understand:

- how
- why
- when
- where

—not simply copy code.

---

## Theory first — before Volume X.

Volumes I–IX teach concepts.

Volume X implements the editor using those previously established concepts and decisions.

Readers should understand the architectural decisions before implementation begins. Within Volume X, the goal is no longer to re-teach those concepts from first principles; the goal is to realize them as working source code and explain how the implementation satisfies the established design.

---

## Chapter independence depends on the volume.

For Volumes I–IX, a reader opening any chapter should be able to understand it with minimal dependence on previous chapters. Cross-references are encouraged.

Volume X is intentionally sequential. Its chapters are implementation milestones in one continuously evolving source tree. Each implementation chapter depends on the source state produced by previous implementation chapters. Repeating previously established theory merely to make an implementation chapter appear standalone should be avoided.

---

# 6. Volume X Implementation Rules

Volume X is intentionally different from Volumes I–IX.

Volumes I–IX establish the theory, architecture, algorithms, engineering practices, and design decisions required to build the editor. Volume X turns those decisions into one complete working terminal text editor.

Therefore, Volume X follows a **code-first implementation workflow**:

```text
Implementation Goal
        ↓
New / Changed Files
        ↓
Complete Source Code
        ↓
Build
        ↓
Run
        ↓
Test
        ↓
Explain the Actual Code
        ↓
Review Architecture and Invariants
        ↓
Next Implementation Milestone
```

The implementation chapters must prioritize the actual source code over theoretical exposition.

For each meaningful implementation milestone:

1. Identify the implementation goal.
2. Identify new and modified files.
3. Provide complete contents of every new or modified source/header file.
4. Keep the complete source state internally consistent.
5. Build the project using its actual CMake configuration.
6. Run or test the implemented behavior.
7. Explain the actual source code that was presented.
8. Connect the implementation to the established architecture and ADRs.
9. Identify important invariants, ownership rules, and failure behavior.
10. Record the resulting implementation state.
11. Continue to the next milestone.

Volume X must not become a second theory volume.

Previously established concepts should normally be referenced rather than re-taught at length. Theory may be explained when it is necessary to understand an implementation, but implementation remains the primary activity.

The source code produced throughout Volume X forms one continuous implementation. Chapters must not create unrelated disposable mini-editors or independent example programs.

The detailed operational rules for Volume X are maintained in the **Implementation Project Context**, which governs the implementation workflow while remaining subordinate to the global book identity, terminology, architecture, coding standards, and established project decisions of this Book Bible.

---

# 7. Writing Style

The writing style should resemble a university textbook combined with a professional software engineering handbook.

Tone:

- Professional
- Educational
- Friendly
- Precise
- Clear
- Concise where appropriate
- Detailed when necessary

Avoid:

- unnecessary humor
- internet slang
- excessive storytelling
- motivational filler
- marketing language

Never assume knowledge that has not yet been introduced.

---

# 8. Explanation Strategy

Every technical topic should follow this progression whenever applicable.

1. Motivation
2. Concept
3. Visual Diagram
4. Simple Example
5. Real-world Analogy (optional)
6. Technical Details
7. Advantages
8. Limitations
9. Summary

---

# 9. Chapter Structure

Unless otherwise specified, every chapter should contain:

- Introduction
- Learning Objectives
- Main Sections
- Notes / Tips
- Common Mistakes
- Summary
- Exercises (where appropriate)
- Preview of the next chapter

---

# 10. Diagram Standards

Diagrams should use ASCII whenever possible.

Example:

+---------+
| Buffer |
+---------+
|
v
+---------+
|Renderer |
+---------+

Use:

- arrows
- boxes
- indentation
- monospace formatting

Avoid image dependencies.

Every diagram should explain something—not merely decorate the page.

---

# 11. Code Philosophy

For Volumes I–IX, code is primarily a teaching artifact. Code examples should illustrate the concept currently being taught without introducing unnecessary implementation complexity.

Every code example should:

- solve one problem
- remain appropriately focused
- be explained
- avoid unnecessary abstraction

For Volume X, this rule changes intentionally: source code is the primary implementation artifact. Complete source files should be shown when they are new or modified as part of an implementation milestone. Large source files must not be artificially split merely to make listings shorter. The reader must be able to reproduce the actual project state from the implementation chapter.

The distinction is:

- **Volumes I–IX:** code illustrates theory.
- **Volume X:** code implements the editor.

---

# 12. Programming Language

Primary language:

C (C99)

Reasons:

- Minimal abstraction
- Direct interaction with POSIX APIs
- Better understanding of systems programming
- Easier transition to C++, Rust, Zig, Go, etc.

No C++ should appear unless explicitly discussed.

---

# 13. Platform

Primary platform:

Linux

Secondary:

macOS

Portability notes:

WSL
Windows
BSD

Platform-specific behavior should be clearly identified.

---

# 14. Project Scope

The implementation targets a Nano-like editor.

Core features:

- open file
- save file
- insert
- delete
- cursor movement
- scrolling
- status bar
- keyboard shortcuts

Advanced features are discussed later.

---

# 15. Terminology Standards

Always use the following terms consistently.

Editor State

Text Buffer

Renderer

Cursor

Viewport

Status Bar

Message Bar

Input Handler

Input Parser

Input Engine

Command Engine

File Manager

Document

Frame

Event Loop

Raw Mode

Canonical Mode

ANSI Escape Sequence

Never invent alternate names for the same concept.

Terminology consistency is critical.

---

# 16. Formatting Rules

Headings should follow:

# Chapter

## Section

### Subsection

Lists should remain short.

Paragraphs should generally remain under six sentences.

Long explanations should be divided using subheadings.

---

# 17. Callout Conventions

Use clearly distinguishable blocks.

### Note

Additional information.

### Tip

Practical advice.

### Warning

Important pitfalls.

### Historical Note

Background information.

### Engineering Insight

Professional design discussion.

---

# 18. Engineering Philosophy

Whenever a design decision is introduced, explain:

- Why
- Alternatives
- Complexity
- Performance
- Memory implications
- Simplicity

In Volumes I–IX, this normally means explaining the design before implementation.

In Volume X, design explanations should be connected to the actual source code being implemented. Do not replace implementation with theoretical discussion when real C code is the appropriate artifact.

---

# 19. Architecture Philosophy

Architecture comes before implementation at the project level.

Volumes I–IX establish:

- responsibilities
- module boundaries
- ownership
- dependencies

Volume X implements that established architecture.

During Volume X, architectural explanations should normally be tied to the implementation milestone and actual source code. Previously established architecture should be referenced rather than re-taught unless the implementation exposes a genuine architectural issue requiring review.

---

# 20. Complexity Discussions

Whenever algorithms are introduced:

Discuss:

Time complexity

Space complexity

Cache locality (when relevant)

Scalability

Trade-offs

Avoid unnecessary mathematical formalism.

Focus on practical engineering.

---

# 21. Progressive Learning

Concepts should be introduced in increasing complexity.

Simple

↓

Practical

↓

Internal Details

↓

Optimization

↓

Professional Techniques

Readers should never encounter advanced concepts before mastering simpler ones.

---

# 22. Code Standards

Use:

snake_case

Meaningful names

Small functions

Minimal globals

Header/source separation

POSIX APIs

Avoid macros unless necessary.

---

# 23. Examples

Examples should progress:

Tiny

↓

Medium

↓

Realistic

Avoid overwhelming readers with complex examples immediately.

---

# 24. Exercises

Where appropriate, exercises should include:

Review questions

Implementation tasks

Thought experiments

Optional challenges

Exercises should reinforce understanding—not introduce new material.

---

# 25. Architecture Decision Records (ADRs)

Major project decisions should be documented.

Examples:

ADR-001

Programming Language

Decision:

C

Reason:

Lowest abstraction.

---

ADR-002

Rendering Strategy

Decision:

Full redraw

Reason:

Simpler implementation.

---

# 26. Cross References

When a chapter builds on previous material, reference it.

Example:

"As discussed in Chapter 5..."

Avoid repeating entire explanations.

---

# 27. Volume Philosophy

Volumes should build naturally.

Volume I

Foundations

↓

Volume II

Architecture

↓

Volume III

Core Engine

↓

Volume IV

Rendering

↓

Volume V

File System

↓

Volume VI

Algorithms

↓

Volume VII

Professional Features

↓

Volume VIII

Engineering

↓

Volume IX

Advanced Topics

↓

Volume X

Implementation

Volumes I–IX establish the theory and design required for implementation. Volume X is the deliberate construction phase. Therefore, theory precedes implementation at the book level, while the chapters of Volume X follow a code-first implementation workflow.

---

# 28. Definition of Success

The book succeeds if readers can:

- explain how a terminal works
- explain raw mode
- explain editor architecture
- implement a basic editor without copying code
- understand why professional editors use different data structures
- confidently explore editor internals beyond this book

The goal is independent understanding—not dependency on the book.

---

# 29. Future Revisions

This document may evolve during writing.

However:

- terminology should remain stable
- educational philosophy should remain stable
- architecture decisions should remain stable

Changes should improve consistency rather than alter direction.

The theory-first philosophy of Volumes I–IX and the code-first implementation philosophy of Volume X are both intentional and should remain distinct.

---

End of Book Bible
