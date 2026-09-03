# 31.14 Limiting Undo History

Undo history can grow for as long as the editor remains open. If every operation is retained indefinitely, a long editing session can eventually consume substantial memory.

The editor therefore needs a **history policy** that determines how much past editing information it retains.

There are several reasonable policies.

## Unlimited History

The simplest policy is to retain every undoable operation until the editing session ends.

```text
Oldest                                      Newest
  │                                            │
  ▼                                            ▼
+----+----+----+----+----+----+----+----+----+
| A  | B  | C  | D  | E  | F  | G  | H  | I  |
+----+----+----+----+----+----+----+----+----+
```

The advantage is predictable behavior: the user can continue undoing as far back as the retained history allows.

The disadvantage is that memory usage has no fixed upper bound.

This is particularly important for operation history because the number of operations alone does not determine memory consumption. A history containing many small edits may be inexpensive, while a small number of large paste or deletion operations may consume significant memory.

## Maximum Number of Operations

Another policy is to limit history by entry count.

For example:

```text
Maximum history = 1000 operations
```

Once the limit is reached, adding a new operation removes the oldest retained entry:

```text
Before:

A B C D E

New operation F:

B C D E F
^
oldest entry removed
```

This provides a simple and predictable bound on the number of history records.

However, it does **not** provide a predictable memory bound.

Consider:

```text
1000 small operations
```

versus:

```text
1000 large paste operations
```

Both contain the same number of entries but can require dramatically different amounts of memory.

## Maximum Memory Budget

A stricter policy is to limit history by memory usage.

For example:

```text
Maximum history memory = 64 MB
```

The history system can remove the oldest entries when adding a new operation would exceed the permitted budget.

Conceptually:

```text
+--------------------------------+
|        History Budget          |
|                                |
| A | B | C | D | E | F | G | H |
+--------------------------------+
^                                ^
oldest                           newest

If a new operation exceeds
the budget:

A is discarded first.
```

This gives the editor a meaningful upper bound on memory consumed by undo/redo.

The trade-off is greater bookkeeping. The history system must know how much memory each retained entry occupies, including operation-specific data such as inserted or deleted text.

## Removing the Oldest History

When history must be reduced, the natural policy is to discard the oldest entries first.

Suppose the history contains:

```text
A B C D E
```

and the oldest entry `A` is removed:

```text
B C D E
```

The user can still undo:

```text
E
D
C
B
```

but cannot continue back to `A`.

This creates a **history boundary**.

The editor has not lost the current Document. It has only discarded the ability to navigate beyond the retained portion of editing history.

This distinction is important:

> History truncation limits how far backward the user can travel; it does not alter the current Document.

## History Truncation and Redo

History limits also interact with the Redo Stack.

Suppose the user has:

```text
Undo:
A B C

Redo:
D E
```

After several undo operations, the user may move entries between the two histories.

If the history policy requires old entries to be discarded, the system must apply the policy consistently to whichever history contains entries that are being retained.

More importantly, creating a new edit after undo still invalidates the Redo history, as established earlier:

```text
Undo
  │
  ▼
New edit
  │
  ▼
Redo history discarded
```

History limits do not change that rule.

They only determine how much history can remain available.

## Choosing a Policy

The three policies represent different priorities:

| Policy             | Strength                | Weakness          |
| ------------------ | ----------------------- | ----------------- |
| Unlimited          | Maximum history         | Unbounded memory  |
| Maximum operations | Simple and predictable  | Poor memory bound |
| Maximum memory     | Controls resource usage | More bookkeeping  |

For a small educational editor, a simple maximum-history policy can be easier to understand and implement. A memory-based limit becomes more attractive when large documents and large editing operations must be handled predictably.

The important architectural decision is to make the policy explicit rather than allowing history growth to become an accidental consequence of implementation.

## Engineering Insight

Undo history is a resource.

Like any other resource, it needs a defined lifetime and, when necessary, a defined limit.

A practical history policy can therefore be expressed as:

```text
Record operation
      │
      ▼
Would history exceed its limit?
      │
   ┌──┴──┐
  No     Yes
   │      │
   │      ▼
   │   Remove oldest
   │   entries
   │      │
   └──┬───┘
      ▼
Continue editing
```

The policy determines how much history the editor promises to retain. Once older entries have been discarded, those earlier editing states are no longer available through undo, but the current Document remains unaffected.

## Summary

Undo history is a resource that needs a defined lifetime and, when necessary, a defined limit. Policies range from unlimited history to a maximum number of operations to a memory budget; when history is reduced, the oldest entries are discarded first, which limits how far backward the user can travel without altering the current Document.

Now that history limits are established, the next section steps back to compare the overall design strategies and their trade-offs.

Continue to **Section 31.15, Design Alternatives and Trade-offs**.
