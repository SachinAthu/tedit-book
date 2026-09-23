# Volume VII — Professional Features

# Chapter 34 — Multiple Buffers

## Introduction

The editor we have built so far works with one editable document at a time. A realistic editing session, however, rarely stays inside a single file. Programmers consult `main.c` while editing `editor.c`, writers move between several documents, and users often want to view two parts of the same file side by side.

Supporting these workflows is not just a matter of storing more text. Once several documents exist, the editor must answer harder questions. Which state belongs to which document? How does the user move between them? When two windows display the same buffer, where does the cursor live? The answers evolve the single-document architecture into a multi-buffer, multi-window one.

Multiple buffers are therefore a **system feature**, not a data structure. This chapter builds the model progressively:

```text
Vocabulary — Document, Buffer, View, Window, Tab
    ↓
Ownership — editor-wide, buffer, and window state
    ↓
Management — the Buffer Manager and lifecycle, switching, and tabs
    ↓
Windows — layout, rendering, and commands
    ↓
Integration — file management
    ↓
Reflection — costs, alternatives, and mistakes
```

We begin by distinguishing the five core concepts, then assign every piece of state — including each existing feature — to the scope that owns it. We design the Buffer Manager and the buffer lifecycle, examine switching and tab organization, and extend layout, rendering, and input to multiple windows. We then apply the model to file management and quantify the costs. We close with the design alternatives and the common mistakes to avoid.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain why a single-document model cannot support realistic editing workflows.
- Distinguish a Document, Buffer, View, Window, and Tab, and state what each one answers.
- Classify state as editor-wide, buffer-specific, or window-specific.
- Describe the Buffer Manager's role in the collection and lifecycle of buffers.
- Explain why switching buffers is a selection operation that preserves each editing context.
- Describe how tabs organize windows without becoming buffers.
- Explain how layout, rendering, and commands operate across multiple windows.
- Describe how file management and the buffer lifecycle work with multiple buffers.
- Assign every existing editor feature a precise scope.
- Evaluate the complexity and resource costs of multiple buffers.
- Compare the design alternatives and recognize common multiple-buffer mistakes.