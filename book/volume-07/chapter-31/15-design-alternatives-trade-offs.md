# 31.15 Design Alternatives and Trade-offs

Undo history can be designed using several strategies. For this editor, the important distinction is between preserving complete Document states and preserving the operations that transform one state into another.

| Strategy          | Strength  | Weakness          |
| ----------------- | --------- | ----------------- |
| Snapshots         | Simple    | High memory usage |
| Operation history | Efficient | More complex      |
| Hybrid            | Flexible  | More complexity   |

## Snapshots

Snapshots are the simplest strategy conceptually.

Each history point preserves a complete Document state:

```text
State A
   │
   ▼
State B
   │
   ▼
State C
```

Undo selects an earlier state rather than reconstructing it from individual operations.

The major advantage is simplicity. The history mechanism does not need to understand how an operation is reversed; it only needs access to the preserved state.

The cost is memory. Large Documents require large snapshots, and retaining many snapshots can consume substantial memory.

## Operation History

Operation history records changes rather than complete Document states:

```text
INSERT "hello"
DELETE "world"
SPLIT_LINE
MERGE_LINE
```

Each operation contains the information required to reverse and reapply the change.

Its major advantage is efficiency. A small edit to a large Document does not require another complete copy of the Document.

The cost is complexity. Every undoable operation must contain sufficient information for both reversal and reapplication, and operations involving large amounts of inserted or deleted text can still consume significant memory.

## Hybrid History

A hybrid approach combines snapshots with operation history.

Conceptually:

```text
Snapshot
   │
   ├── Operation
   ├── Operation
   ├── Operation
   │
Snapshot
   │
   ├── Operation
   ├── Operation
   └── Operation
```

Instead of retaining every complete state, the system periodically establishes a snapshot and records operations between snapshots.

This can provide a balance between memory usage and the complexity of reconstructing older states.

However, the additional mechanism also introduces additional design decisions:

- when to create snapshots
- how many operations to retain between snapshots
- how snapshots and operations interact
- how memory should be allocated between the two

For the baseline editor, that additional complexity is not necessary.

## The Architectural Choice

The three strategies can therefore be viewed as points on a design spectrum:

```text
Simpler                                      More flexible

Snapshots ───────── Operation History ───────── Hybrid
    │                     │                      │
    │                     │                      │
More memory          More logic             More machinery
```

The project prioritizes simplicity and educational value while still requiring a useful undo/redo system. Operation history provides a practical balance: it avoids copying the entire Document for ordinary edits while keeping history conceptually tied to the operations executed by the Command Engine.

The important conclusion is not that one strategy is universally superior.

> The appropriate undo strategy depends on the editor's workload, memory constraints, and desired implementation complexity.

For this project, the architecture should preserve the separation between the **Document**, **Command Engine**, and **History** regardless of the underlying history strategy. That keeps the history mechanism replaceable without requiring unrelated components to understand its internal representation.

## Summary

Undo history can be built from snapshots, operation records, or a hybrid of the two. Snapshots are simplest but memory-hungry, operation history is efficient but requires each command to carry reversal information, and a hybrid balances the two at the cost of extra machinery.

For this editor, operation history provides the practical balance. Now that the strategies have been compared, the next section examines the common bugs that undo/redo implementations tend to introduce.

Continue to **Section 31.16, Common Undo / Redo Bugs**.
