# 31.7 The Redo Stack

The undo stack answers one question:

> What operation can be undone next?

The redo stack answers the opposite question:

> What operation can be reapplied next?

It is therefore the second half of the undo/redo history model.

```text
                 Editor State
                     │
                     ▼
              ┌──────────────┐
              │  Undo Stack  │
              └──────────────┘
                     │
                  undo
                     ▼
              ┌──────────────┐
              │  Redo Stack  │
              └──────────────┘
                     │
                  redo
                     │
                     └──────────► Undo Stack
```

## Moving an Operation to Redo

When an operation is undone, it should not simply be discarded.

Instead, it moves from the undo stack to the redo stack.

```text
Before Undo:

Undo: [A, B, C]
Redo: []

        undo C
           │
           ▼

After Undo:

Undo: [A, B]
Redo: [C]
```

The operation remains available because the user may decide that the undo was itself unwanted.

Pressing Redo then moves the operation back:

```text
Before Redo:

Undo: [A, B]
Redo: [C]

        redo C
           │
           ▼

After Redo:

Undo: [A, B, C]
Redo: []
```

This movement is more accurate than thinking of redo as creating a new operation. The same history entry is being moved between the two histories.

## Redo Reapplies the Original Operation

Suppose the user inserts `"hello"`:

```text
INSERT_TEXT
position: (2, 5)
text: "hello"
```

Undo removes that text.

Redo should then perform the original insertion again:

```text
Undo:
    delete "hello"

Redo:
    insert "hello"
```

The redo operation is therefore based on the original command, not on the inverse command that was used to undo it.

Conceptually:

```text
Original:
    INSERT_TEXT

Undo:
    inverse(INSERT_TEXT)
    → DELETE_TEXT

Redo:
    INSERT_TEXT
```

This distinction is important because the inverse operation exists to move backward through history, while the original operation moves forward again.

## Redo Is Invalidated by a New Edit

The most important rule governing the redo stack is what happens after an undo.

Consider:

```text
A → B → C
```

After undoing `C`:

```text
A → B
```

The history is:

```text
Undo: [A, B]
Redo: [C]
```

The user can still redo `C`.

But suppose the user now performs a new operation `D`:

```text
A → B → D
```

The redo history must be cleared:

```text
Undo: [A, B, D]
Redo: []
```

Why?

Because `C` belongs to the previous editing path. Reapplying it after `D` would produce a history that the user did not create through the normal sequence of operations.

```text
Original path:

A → B → C


New path:

A → B → D
```

The conventional two-stack model does not preserve both branches.

> **Once a new edit is made after Undo, the redo stack is discarded.**

## Undo and Redo Form a Pair

The two stacks should therefore be treated as parts of one history mechanism.

A normal edit performs:

```text
New Edit
   │
   ├── push onto Undo
   └── clear Redo
```

Undo performs:

```text
Undo
   │
   ├── pop from Undo
   ├── apply inverse
   └── push onto Redo
```

Redo performs:

```text
Redo
   │
   ├── pop from Redo
   ├── apply original operation
   └── push onto Undo
```

This gives the history system three fundamental transitions:

```text
                 New Edit
                    │
                    ▼
              ┌───────────┐
              │ Undo Stack│
              └───────────┘
                    │
                  undo
                    ▼
              ┌───────────┐
              │ Redo Stack│
              └───────────┘
                    │
                  redo
                    │
                    └──────► Undo Stack
```

## Why Redo Should Not Be a Separate History

A common mistake is to think of redo as another independent collection of operations.

That creates unnecessary complexity.

Undo and redo are really two views of the same linear editing history:

```text
       Earlier                    Later
          │                         │
          ▼                         ▼

    [ A ][ B ][ C ][ D ][ E ]
                    ▲
              current position
```

Operations to the left of the current position have already been performed and may be undoable.

Operations to the right have been undone and may be redoable.

The two stacks provide a simple way to represent that position without maintaining a more complicated history structure.

## Empty Stacks

Both stacks can be empty.

If the undo stack is empty:

```text
Undo: []
Redo: [A, B]
```

there is nothing left to undo.

If the redo stack is empty:

```text
Undo: [A, B]
Redo: []
```

there is nothing available to redo.

These are normal states, not error conditions.

The editor should simply treat the corresponding command as having no effect, while potentially reporting appropriate user feedback through the Message Bar if that matches the editor's command behavior.

## The Central Rule

The redo stack is therefore not an archive of old operations. It represents the **currently undone portion of the linear editing history**.

Its lifecycle is simple:

```text
New Edit
   ↓
Redo cleared

Undo
   ↓
Operation moves to Redo

Redo
   ↓
Operation moves back to Undo

New Edit after Undo
   ↓
Redo cleared
```

This simple rule provides predictable behavior while keeping the history model small and understandable.

## Summary

The redo stack holds the undocument portion of the editing history—operations that were undone and can be reapplied. Undo moves an operation from the undo stack to the redo stack, redo moves it back, and a new edit after an undo discards the redo stack so history remains a single linear path.

Now that both stacks are defined, the next section presents undo and redo together as a state machine.

Continue to **Section 31.8, Undo / Redo State Machine**.
