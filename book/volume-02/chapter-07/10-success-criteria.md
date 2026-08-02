# 7.10 Success Criteria

At this point, our Software Requirements Specification is nearly complete.

We have defined:

- the scope of the project,
- the functional requirements,
- the non-functional requirements,
- the project constraints,
- the operating assumptions, and
- the primary use cases.

One important question still remains:

> **How do we know when the project is finished?**

Without a clear answer, software development can continue indefinitely. There is always another feature to add, another optimization to make, or another improvement to consider.

To avoid this, every project should define a set of **success criteria**.

Success criteria establish the measurable conditions that determine whether the project's objectives have been achieved.

## What Are Success Criteria?

**Success criteria** are objective standards used to evaluate whether the completed software satisfies its intended goals.

Unlike individual requirements, which describe specific behaviors, success criteria evaluate the system as a whole.

For example:

A functional requirement might state:

> The editor shall save the current document.

A success criterion might state:

> Users can reliably edit and save text documents without data loss.

The requirement describes one capability.

The success criterion evaluates whether the completed system successfully fulfills its overall purpose.

## Why Success Criteria Matter

Imagine building software without defining what success looks like.

Even after months of development, questions continue to arise:

- Should we add syntax highlighting?
- Should we support multiple documents?
- Should we improve rendering performance?
- Should we implement plugins?

None of these ideas are necessarily bad.

The problem is that without a clearly defined endpoint, the project never truly reaches completion.

Success criteria provide that endpoint.

They answer an important engineering question:

> **"When have we built enough?"**

Once every success criterion has been satisfied, the project has achieved the objectives established in its scope.

Future improvements become enhancements rather than unfinished work.

## Success Criteria for Our Editor

Our editor will be considered successful if it satisfies the following objectives.

### SC-001 — Document Editing

A user can create a new document, open an existing text file, modify its contents, and save the result without data loss. This end-to-end editing workflow functions correctly using only the editor's built-in commands.

This represents the editor's primary purpose.

---

### SC-002 — Reliable Keyboard Interaction

Every supported key produces its intended action during an editing session. Printable characters appear at the cursor, navigation keys reposition the cursor correctly, and keyboard shortcuts invoke the corresponding commands.

Users can interact with the editor entirely through the keyboard.

---

### SC-003 — Correct Screen Rendering

The terminal display accurately reflects the current editor state after every user action. The document text, cursor position, status bar, and messages remain synchronized without visual artifacts.

---

### SC-004 — Stable File Operations

Files are loaded and saved correctly through repeated edit-and-save cycles. When a file operation fails, the editor reports the error clearly and continues running rather than terminating.

Protecting user data is one of the most important responsibilities of any text editor.

---

### SC-005 — Responsive User Experience

Keyboard input results in a screen update with no perceptible delay during normal editing. The editor remains responsive when opening, editing, and navigating files of typical size.

---

### SC-006 — Maintainable Architecture

A developer unfamiliar with the codebase can identify the major modules, understand their responsibilities, and trace a user action from input to screen update within a brief review of the source structure.

This criterion reflects the educational objective of the project. The editor is intended not only to function correctly but also to demonstrate professional software design practices.

## Measuring Success

One advantage of clearly defined success criteria is that they are measurable.

Consider the following checklist.

```text
Project Evaluation

✓ Opens files

✓ Saves files

✓ Edits text

✓ Moves cursor

✓ Displays correctly

✓ Handles keyboard input

✓ Maintains document state

✓ Follows architectural design
```

When every item has been verified, we can confidently say that the editor fulfills its intended purpose.

Notice that this checklist does **not** include advanced features such as:

- syntax highlighting,
- plugins,
- multiple documents,
- code completion,
- collaborative editing.

These capabilities lie outside the project's scope and therefore are **not** part of its success criteria.

This reinforces an important engineering principle:

> **Success is measured against the project's defined objectives—not against every feature that could possibly be added.**

## Success Is Relative to Scope

It is important to remember that success criteria are always tied to the project's scope.

For example:

```text
Project Scope

↓

Requirements

↓

Architecture

↓

Implementation

↓

Success Evaluation
```

If the scope changes, the success criteria change as well.

For our educational editor, success means building a reliable, maintainable, terminal-based text editor that demonstrates the core principles of systems programming and software architecture.

Professional editors such as Vim, Emacs, or VS Code naturally have much broader success criteria because they solve a much larger set of problems.

## The Final SRS Document

With the success criteria defined, we have now explored every component of the Software Requirements Specification:

- What problem are we solving?
- What features will the editor provide?
- What qualities should it possess?
- What limitations exist?
- What assumptions are being made?
- How will users interact with it?
- How will we know when the project is complete?

In the next section, all of these elements are compiled into a single, complete Software Requirements Specification document. This serves as the authoritative reference that will guide every architectural decision throughout the remainder of this book.

## Engineering Insight

Many software projects struggle not because they lack talented developers, but because they lack a clear definition of success. When success criteria are established early, design decisions become easier, priorities become clearer, and development remains focused on achieving well-defined objectives rather than endlessly adding features.

## Summary

Success criteria define the measurable conditions that determine whether the project has achieved its intended goals. They provide a clear endpoint for development, ensure that the implementation remains aligned with the project's scope, and establish objective standards by which the completed editor can be evaluated.

With every element of the Software Requirements Specification now examined, the next section brings them together into a single reference document: the **complete SRS** for our terminal text editor.
