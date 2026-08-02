# Building a Terminal Text Editor

## Book Bible (Project Constitution)

**Version:** 1.0  
**Status:** Active  
**Applies to:** Every volume, chapter, diagram, example, and code sample.

---

# 1. Purpose

This document defines the writing standards, educational philosophy, terminology, formatting rules, architectural decisions, and project conventions for the book.

Its purpose is to ensure the entire book feels as though it was written by a single author with a consistent teaching style and technical depth.

Whenever a future chapter conflicts with this document, **this document takes precedence**.

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

## Explain before implementing.

Every concept should answer:

1. What problem exists?
2. Why does it exist?
3. What solutions exist?
4. Why was this solution chosen?
5. What are the trade-offs?

Only then should implementation begin.

---

## Build understanding, not memorization.

Readers should understand:

- how
- why
- when
- where

—not simply copy code.

---

## Theory first.

Volumes I–IX teach concepts.

Volume X implements the editor.

Readers should understand every architectural decision before writing code.

---

## Every chapter should stand on its own.

A reader opening any chapter should be able to understand it with minimal dependence on previous chapters.

Cross-references are encouraged.

---

# 6. Writing Style

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

# 7. Explanation Strategy

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

# 8. Chapter Structure

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

# 9. Diagram Standards

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

# 10. Code Philosophy

Code exists to teach concepts.

It is not the primary focus.

Every code example should:

- solve one problem
- remain short
- be explained
- avoid unnecessary abstraction

Large code listings should be split into smaller pieces.

---

# 11. Programming Language

Primary language:

C (C99)

Reasons:

- Minimal abstraction
- Direct interaction with POSIX APIs
- Better understanding of systems programming
- Easier transition to C++, Rust, Zig, Go, etc.

No C++ should appear unless explicitly discussed.

---

# 12. Platform

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

# 13. Project Scope

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

# 14. Terminology Standards

Always use the following terms consistently.

Editor State

Text Buffer

Renderer

Cursor

Viewport

Status Bar

Message Bar

Input Handler

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

# 15. Formatting Rules

Headings should follow:

# Chapter

## Section

### Subsection

Lists should remain short.

Paragraphs should generally remain under six sentences.

Long explanations should be divided using subheadings.

---

# 16. Callout Conventions

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

# 17. Engineering Philosophy

Whenever a design decision is introduced:

Explain:

- Why
- Alternatives
- Complexity
- Performance
- Memory implications
- Simplicity

Never present implementation without discussing design decisions.

---

# 18. Architecture Philosophy

Architecture before implementation.

Readers should first understand:

- responsibilities
- module boundaries
- ownership
- dependencies

Implementation comes afterwards.

---

# 19. Complexity Discussions

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

# 20. Progressive Learning

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

# 21. Code Standards

Use:

snake_case

Meaningful names

Small functions

Minimal globals

Header/source separation

POSIX APIs

Avoid macros unless necessary.

---

# 22. Examples

Examples should progress:

Tiny

↓

Medium

↓

Realistic

Avoid overwhelming readers with complex examples immediately.

---

# 23. Exercises

Where appropriate, exercises should include:

Review questions

Implementation tasks

Thought experiments

Optional challenges

Exercises should reinforce understanding—not introduce new material.

---

# 24. Architecture Decision Records (ADRs)

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

# 25. Cross References

When a chapter builds on previous material, reference it.

Example:

"As discussed in Chapter 5..."

Avoid repeating entire explanations.

---

# 26. Volume Philosophy

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

Theory always precedes implementation.

---

# 27. Definition of Success

The book succeeds if readers can:

- explain how a terminal works
- explain raw mode
- explain editor architecture
- implement a basic editor without copying code
- understand why professional editors use different data structures
- confidently explore editor internals beyond this book

The goal is independent understanding—not dependency on the book.

---

# 28. Future Revisions

This document may evolve during writing.

However:

- terminology should remain stable
- educational philosophy should remain stable
- architecture decisions should remain stable

Changes should improve consistency rather than alter direction.

---

End of Book Bible
