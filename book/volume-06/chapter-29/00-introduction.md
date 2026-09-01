# Volume VI — Algorithms & Data Structures

# Chapter 29 — Comparing Text Buffers

## Introduction

## Learning Objectives

### 29.1 — From Data Structures to Real Editors

A very short introduction.

Explain that there is a difference between:

> "Which data structure is theoretically better?"

and:

> "Which data structure did an actual editor choose for its requirements?"

Introduce the seven editors.

---

### 29.2 — Nano

Focus:

- What text representation does Nano use?
- Why is that appropriate for a small terminal editor?
- What does this tell us about simplicity?

Keep it practical and brief.

---

### 29.3 — Vim

Focus:

- How Vim represents and edits text
- Why its architecture evolved differently from a simple line-based editor
- How its editing model influences its buffer design

Avoid diving into Vim's entire architecture.

---

### 29.4 — Emacs

Focus:

- Emacs's buffer model
- How its requirements differ from Nano/Vim
- Why extensibility and long-lived buffers influence its design

Again, only discuss the parts relevant to **text storage**.

---

### 29.5 — VS Code

Focus:

- VS Code's text-buffer architecture
- Piece-oriented text representation
- Why large-scale editing and undo/history influence the design

This is where the reader sees a much more sophisticated approach than our editor.

---

### 29.6 — Helix and Zed

I would combine these because the goal is comparison rather than seven independent mini-chapters.

Discuss:

**Helix**

- modern editor workload
- text representation
- relationship with syntax-aware editing

**Zed**

- modern high-performance editor
- text-buffer architecture
- performance/concurrency pressures

The point is to show that modern editors have substantially different requirements from a Nano-like educational editor.

---

### 29.7 — Sublime Text

Focus on:

- performance-oriented design
- proprietary implementation
- what can and cannot be confidently inferred about its internal text representation

This is important because we should **not pretend to know proprietary implementation details that are not publicly documented**.

---

### 29.8 — Which Editor Uses What?

This becomes the chapter's main reference table.

Something along these lines:

| Editor           | Text representation / approach                        | Why it matters                            |
| ---------------- | ----------------------------------------------------- | ----------------------------------------- |
| **Nano**         | Line-oriented/simple buffer                           | Small, straightforward terminal editor    |
| **Vim**          | Specialized buffer representation                     | Efficient interactive editing             |
| **Emacs**        | Specialized buffer model                              | Extensibility and sophisticated editing   |
| **VS Code**      | Piece-oriented text representation                    | Efficient large-scale editing and history |
| **Helix**        | Modern text-buffer architecture                       | Syntax-aware, high-performance editing    |
| **Zed**          | Modern high-performance text model                    | Responsiveness and large-editor workloads |
| **Sublime Text** | Proprietary / implementation details not fully public | Performance-oriented editor               |

We should verify the exact implementation claims before writing this section, especially for **Vim, Emacs, Helix, Zed, and Sublime Text**.

---

### 29.9 — What This Means for Our Editor

This should be the conclusion rather than another generic trade-off chapter.

Our editor has deliberately different requirements:

```text
Our Editor

Small
   +
Single document
   +
Educational
   +
C99
   +
Simple architecture
   +
Entire document in memory
        │
        ▼
Simple text representation
```

Therefore, we **do not need to imitate VS Code, Zed, or another modern editor**.

Our existing ADR-013 decision remains appropriate: the project uses a **vector of lines** because its requirements prioritize simplicity and educational value.

The important lesson is:

> **Real editors don't choose the same text buffer. They choose representations that fit their requirements, workloads, and architecture.**

---

# Final Chapter Structure

I would therefore reduce Chapter 29 to:

```text
29.1 — From Data Structures to Real Editors

29.2 — Nano

29.3 — Vim

29.4 — Emacs

29.5 — VS Code

29.6 — Helix and Zed

29.7 — Sublime Text

29.8 — Which Editor Uses What?

29.9 — What This Means for Our Editor

29.10 — Summary

29.11 — Exercises
```
