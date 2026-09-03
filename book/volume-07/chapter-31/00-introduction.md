# Volume VII — Professional Features

# Chapter 31 — Undo / Redo

## Introduction

Every user makes mistakes. A terminal text editor that cannot reverse an edit quickly becomes fragile: one wrong keystroke can destroy work the user never intended to lose. Undo and redo quietly turn the editor from a tool that applies changes into a tool the user can safely experiment with.

Undo appears simple at first—"put the document back"—but the editor must answer harder questions. How much information does history need to retain? Should it store complete states or the changes themselves? Which commands deserve a history entry at all? What happens to redo when the user edits again?

Undo is therefore a **system feature**, not a single data structure. It builds on the architecture established earlier in the book: the Command Engine executes commands, and the undo system retains enough information to reverse the commands that modified the Document.

This chapter builds our understanding of undo and redo progressively:

```text
Problem — what must history remember?
    ↓
Strategy — snapshots vs. operations
    ↓
Model — the two-stack state machine
    ↓
Integration — cursor, dirty flag, grouping
    ↓
Engineering — memory, limits, and bugs
```

We first clarify what an undo operation means, then compare snapshot-based history with operation-based history. We examine how undo and redo form a two-stack state machine, decide which actions belong in history, and integrate undo with cursor state and the Dirty Flag. Finally, we study grouping and coarse coalescing, memory and performance, history limits, design alternatives, and the bugs that undo/redo implementations commonly introduce.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain why undo requires that the editor retain information about past changes.
- Describe what an undo operation means and what an undo record must contain.
- Distinguish snapshot-based history from operation-based history and their trade-offs.
- Explain how undo and redo stacks form a state machine, and why new edits invalidate redo history.
- Identify which commands should and should not enter the undo history.
- Explain when undo must also restore cursor position and editor state.
- Describe how undo and redo relate to the Dirty Flag.
- Explain how grouping and coalescing shape history at user-action granularity.
- Evaluate the memory and performance costs of the baseline undo design.
- Compare history-limiting policies and common undo/redo bugs.