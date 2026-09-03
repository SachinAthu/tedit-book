# 31.8 Undo / Redo State Machine

Undo and redo are easiest to reason about when history is treated as a **state machine** rather than as two unrelated features.

At any point, the editor has a current document state and a history of operations that can either be undone or reapplied.

The fundamental model uses two stacks:

```text
                 Current State
                     │
                     ▼
              ┌───────────────┐
              │ Undo History  │
              └───────────────┘
                     │
                     │ undo
                     ▼
              ┌───────────────┐
              │ Redo History  │
              └───────────────┘
                     │
                     │ redo
                     └──────────────►
```

The two stacks represent different possibilities:

- **Undo stack** — operations that have already been performed and can be reversed.
- **Redo stack** — operations that were undone and can potentially be performed again.

## Performing a New Operation

When the user performs a new undoable operation, it is added to the undo history.

The redo history is cleared because the user has created a new editing path.

```text
Before:

Undo Stack:  [A, B]
Redo Stack:  [C]

             New Operation D
                    │
                    ▼

After:

Undo Stack:  [A, B, D]
Redo Stack:  []
```

This behavior is essential.

Suppose the user performs:

```text
A → B → C
```

Then undoes twice:

```text
A → B
```

At this point, `C` is available for redo.

But if the user now performs a different operation `D`:

```text
A → B → D
```

there is no longer a single linear history in which `C` can be reapplied after `D`.

Therefore:

> **A new edit after an undo invalidates the redo history.**

This is one of the most important rules in a conventional undo/redo system.

## Undoing an Operation

When the user requests undo, the most recent operation is taken from the undo stack.

Conceptually:

```text
Undo Stack                 Redo Stack

[A, B, C]                  []

     │
     │ undo C
     ▼

[A, B]                     [C]
```

The command's inverse operation is applied to the document.

The command then moves from the undo history to the redo history.

The process is therefore:

```text
1. Take the latest undo record.
2. Reverse its effect.
3. Move the record to the redo stack.
```

The history records themselves do not disappear. They change which direction of traversal is currently possible.

## Redoing an Operation

Redo performs the opposite transition.

Starting with:

```text
Undo Stack:  [A, B]
Redo Stack:  [C]
```

a redo operation reapplies `C`:

```text
Undo Stack:  [A, B, C]
Redo Stack:  []
```

Conceptually:

```text
1. Take the latest redo record.
2. Reapply its forward operation.
3. Move the record back to the undo stack.
```

This creates a simple cycle:

```text
             ┌──────────────┐
             │              │
             ▼              │
        Undo Operation      │
             │              │
             ▼              │
        Redo History        │
             │              │
             │ redo         │
             ▼              │
        Undo History ───────┘
```

## The Three Fundamental Transitions

The complete model can be reduced to three important transitions.

### New Edit

```text
Undo Stack:  U
Redo Stack:  R

new operation X

Undo Stack:  U + X
Redo Stack:  empty
```

### Undo

```text
Undo Stack:  U + X
Redo Stack:  R

undo

Undo Stack:  U
Redo Stack:  R + X
```

### Redo

```text
Undo Stack:  U
Redo Stack:  R + X

redo

Undo Stack:  U + X
Redo Stack:  R
```

These transitions define the core behavior of the history system.

## Example

Consider a user making three edits:

```text
Type "A"
Type "B"
Type "C"
```

The history becomes:

```text
Undo: [A, B, C]
Redo: []
```

The user presses Undo:

```text
Undo: [A, B]
Redo: [C]
```

They press Undo again:

```text
Undo: [A]
Redo: [C, B]
```

They now press Redo:

```text
Undo: [A, B]
Redo: [C]
```

Finally, they type `"D"`:

```text
Undo: [A, B, D]
Redo: []
```

The previous `"C"` operation is no longer redoable.

## Why a State Machine Helps

Thinking in terms of state transitions prevents several common design errors.

For example, an implementation might accidentally leave an old redo entry after a new edit:

```text
Undo: [A, B, D]
Redo: [C]       ← incorrect
```

That would allow the editor to attempt to reapply `C` even though the document has followed a different editing path.

The state-machine model makes the required transition explicit:

```text
New Edit
   │
   ├── add to Undo
   └── clear Redo
```

Similarly, undo and redo should **move history records between stacks**, rather than independently creating unrelated records.

## History Is Linear

This model represents a **linear editing history**.

After:

```text
A → B → C
```

and undoing `C`, the editor is effectively at:

```text
A → B
      \
       C
```

If the user creates `D`, the conventional model chooses the new path:

```text
A → B → D
```

rather than maintaining both branches.

A more sophisticated editor could theoretically preserve branching histories, but that is a substantially different history model and is outside the scope of this editor.

The two-stack model is valuable because it provides predictable behavior with relatively little state.

> **Undo and redo are best understood as controlled movement through a linear history: new edits extend the undo history, undo moves operations toward redo, and a new edit after undo discards the redo path.**

## Summary

Treating undo and redo as a state machine keeps the history model simple and correct. The system has just three fundamental transitions—new edit, undo, and redo—and making them explicit prevents common errors such as leaving stale redo entries or treating undo as a new user edit.

Now that the state machine is established, the next section addresses which commands should enter the history at all.

Continue to **Section 31.9, What Counts as an Undoable Action?**.
