# 31.16 Common Undo / Redo Bugs

Undo and redo are deceptively easy to describe and surprisingly easy to implement incorrectly. The basic model may be simple, but small inconsistencies between the Document, history, Cursor, and Dirty Flag can produce difficult-to-diagnose failures.

The most useful way to understand these problems is to examine the assumptions that can become inconsistent.

## Forgetting to Clear Redo After a New Edit

One of the most common errors is allowing old redo entries to survive after a new editing operation.

Consider:

```text
Edit A
  ↓
Edit B
  ↓
Undo B
```

The user is now at the state after Edit A.

If the user performs a new edit:

```text
Edit C
```

the previous future represented by Edit B is no longer valid.

The history must therefore reflect:

```text
Edit A → Edit C
```

rather than:

```text
Edit A → Edit B
         ↘
           Edit C
```

Allowing the user to redo B after creating C would attempt to follow a history branch that no longer corresponds to the current Document.

## Recording Undo as a New Edit

Undo itself changes the Document, but it must not be recorded as an ordinary new user edit.

A common mistake is:

```text
Undo
  ↓
Document changes
  ↓
Record new history entry
```

This can cause history to grow while undoing, potentially producing behavior such as:

```text
Undo
Undo
Undo
...
```

where each undo creates another operation that itself must be undone.

Undo and redo are **history operations**. They apply existing history entries; they do not create new user-edit history entries in the ordinary way.

## Losing Deleted Text

Deletion is especially vulnerable to incomplete history records.

Suppose:

```text
Before:

hello world|

After:

hello |
```

The deleted text must remain available:

```text
DELETE
Text: "world"
Position: ...
```

Without that information, undo cannot reconstruct the previous Document.

The same principle applies to more complex operations such as line splitting and merging. An operation record must preserve everything required to reverse the specific change.

## Incorrect Operation Ordering

The order of operations matters.

Consider:

```text
INSERT A
INSERT B
INSERT C
```

Undo must reverse them in the opposite order:

```text
Undo C
Undo B
Undo A
```

Reversing operations in the wrong order can produce an invalid Document.

The same problem can occur inside a grouped operation. If several low-level changes belong to one user action, their reversal must preserve the dependencies between those changes.

## Incorrect Cursor Restoration

An operation can restore the Document correctly while leaving the Cursor in the wrong position.

For example:

```text
Before:
hello|

After:
hello world|

Undo:
hello world
        ^
```

The Document is correct, but the Cursor is not.

The operation history therefore needs sufficient cursor information to restore the meaningful editing position associated with the operation, as discussed in Section 31.10.

## Incorrect Grouping or Coalescing

Grouping introduces another class of bugs.

If five character insertions are intended to represent one typing action:

```text
INSERT h
INSERT e
INSERT l
INSERT l
INSERT o
```

but the history system accidentally creates two groups:

```text
INSERT "hel"
INSERT "lo"
```

then:

```text
UNDO
```

will produce behavior that does not match the intended user experience.

Conversely, grouping unrelated operations together can make undo too coarse.

The history boundary must therefore correspond to the intended user-visible action.

## Dirty Flag Inconsistencies

The Dirty Flag can also become inconsistent with the Document.

For example:

```text
Saved:
hello

Edit:
hello world
Dirty = true

Undo:
hello
```

If the editor leaves:

```text
Dirty = true
```

then it incorrectly reports unsaved changes.

The opposite problem is equally dangerous:

```text
Saved:
hello

Edit:
hello world
Undo only part of the edit:
hello wor

Dirty = false   ← incorrect
```

The Dirty Flag must reflect whether the current Document differs from the saved version, rather than simply whether undo or redo was performed.

## History Entry Does Not Match the Actual Change

A particularly subtle failure occurs when the history record describes a different operation from what actually happened.

Suppose the editor records:

```text
INSERT "abc"
```

but the Document was actually modified to contain:

```text
INSERT "abd"
```

Undo will attempt to remove or reverse the wrong content.

The result is a mismatch between:

```text
History
   │
   ▼
Expected Document transition
```

and:

```text
Actual Document transition
```

History is only reliable when these two transitions agree.

This is why an operation record should be created from the actual information associated with the successful editing operation rather than from assumptions about what the operation was supposed to do.

## Undo After a Failed Operation

A failed editing operation should not create a history entry.

Consider:

```text
Attempt edit
     │
     ▼
Operation fails
     │
     ▼
Document unchanged
```

There is nothing to undo.

If a history entry is recorded anyway:

```text
Failed edit
    ↓
History entry
    ↓
Undo
```

the history no longer represents actual Document changes.

The same principle applies to file operations. A failed save does not constitute a successful document transition and should not be represented as an undoable editing operation.

## C Memory Ownership Errors

In C, history introduces additional ownership responsibilities.

Operation records may contain dynamically allocated data such as deleted text.

For example:

```text
History Entry
+--------------------+
| Operation metadata |
| Deleted text ------|----> allocated memory
+--------------------+
```

The implementation must establish clearly:

- who owns the allocated text
- when ownership is transferred
- when the memory is freed
- whether undo and redo reuse or copy the data
- what happens when a history entry is discarded

A failure to define ownership can cause:

- memory leaks
- double frees
- dangling pointers
- use-after-free
- corrupted history

These are especially dangerous because the resulting failure may occur long after the original edit.

## Engineering Insight

Most undo/redo bugs are not failures of the two-stack idea itself. They are failures of **consistency between history and the rest of the editor**.

A useful invariant is:

```text
History record
      │
      ▼
must accurately describe
the corresponding Document change
      │
      ├── Cursor state must remain consistent
      ├── Dirty Flag must remain consistent
      └── Memory ownership must remain valid
```

The history system is therefore not an isolated feature. It participates in the Command Engine's state transitions and must preserve the same architectural boundaries and ownership rules as the rest of the editor.

## Summary

Most undo/redo bugs are not failures of the two-stack model itself; they are failures of consistency between history and the rest of the editor. A history record must accurately describe the corresponding Document change, and the Cursor, Dirty Flag, and memory ownership must all remain consistent with the actual editing state.

Continue to the **Chapter Summary**, which consolidates the undo/redo design, followed by exercises and a preview of the next chapter.
