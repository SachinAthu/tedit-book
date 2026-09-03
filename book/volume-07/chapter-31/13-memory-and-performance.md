# 31.13 Memory and Performance

Undo and redo introduce additional memory and processing costs because the editor must retain information about previous editing operations.

For a small document, these costs may be negligible. As documents and histories grow, however, the choice between snapshots and operation history becomes significant.

## Snapshot History

With snapshots, each history entry contains a representation of the Document at a particular point in time.

Conceptually:

```text
Snapshot 1 → complete Document
Snapshot 2 → complete Document
Snapshot 3 → complete Document
```

If the Document contains a large amount of text, every snapshot can represent a substantial memory allocation.

For a document of size `N`, a snapshot is approximately proportional to:

```text
O(N)
```

per stored document state.

A history containing `H` snapshots can therefore approach:

```text
O(H × N)
```

memory in the simplest model.

This is the primary weakness of snapshots: a small editing action can cause a large amount of information to be retained.

## Operation History

Operation history stores the change rather than another complete Document.

For example:

```text
INSERT "hello"
```

requires only the information necessary to describe and reverse that insertion.

A deletion may need to retain the deleted text:

```text
DELETE "world"
```

The memory required by an operation is therefore related more closely to the amount of changed data than to the total size of the Document.

Conceptually:

```text
Snapshot:

Document size
████████████████████


Operation:

Changed region
████
```

For small edits to large documents, this can provide a substantial reduction in memory usage.

However, operation history does **not** guarantee tiny history entries. A single paste operation might insert several megabytes of text:

```text
PASTE [large text]
```

The history must retain that inserted data if it is required to undo the operation.

Therefore, operation history should be thought of as **change-sized**, not necessarily **constant-sized**.

## Large Insertions and Deletions

The amount of text associated with an operation can dominate history memory.

For example:

```text
DELETE [1 MB of text]
```

cannot be undone correctly unless the deleted text remains available somewhere.

Similarly:

```text
INSERT [1 MB of text]
```

may require the operation history to retain enough information to remove that inserted content during undo and reapply it during redo.

This gives operation history a practical memory characteristic:

```text
History memory
≈
operation metadata
+
stored inserted/deleted data
```

The actual cost depends on the operations performed, not merely the number of history entries.

## Copying Costs

Snapshots can also impose copying costs when a new history entry is created.

A complete Document may need to be copied:

```text
Current Document
       │
       │ copy
       ▼
New Snapshot
```

For a large Document, this can make each snapshot operation expensive.

Operation history generally avoids copying unrelated document content. An insertion record, for example, does not need another copy of every line that was unaffected by the insertion.

The distinction is particularly important for an interactive editor because history recording occurs during user interaction. Extra copying can increase the latency of an editing command.

## Allocation Overhead

Both strategies can involve dynamic memory allocation.

Snapshot history may allocate:

- the snapshot itself
- each stored line
- supporting structures for the copied Document

Operation history may allocate:

- history records
- operation-specific data
- stored inserted or deleted text
- grouping metadata

The allocation pattern is therefore different.

```text
Snapshots
    ↓
Large, document-oriented allocations

Operations
    ↓
Many smaller, operation-oriented allocations
```

Neither pattern is universally faster.

Frequent small allocations can introduce allocator overhead and fragmentation, while large snapshot copies can consume substantial memory bandwidth.

The appropriate design depends on the workload and implementation.

## Cache Behavior

Memory layout can also influence performance.

Snapshot creation involves traversing and copying large portions of the Document. This can touch substantial amounts of memory even when the user's edit is very small.

Operation history usually touches the changed data and the history structures associated with that operation.

For example:

```text
Large Document
+--------------------------------------+
|                                      |
|        mostly unchanged text         |
|                                      |
|              [edit]                  |
|                                      |
+--------------------------------------+

Snapshot:
    potentially touches the entire document

Operation history:
    primarily records the affected region
```

This does not mean operation history is automatically cache-efficient. Its data may be distributed across multiple allocations, and the exact behavior depends on the representation. The important architectural difference is that snapshots encourage whole-document copying, whereas operation history can localize work around the changed data.

## History Size

History size should be considered independently from Document size.

A small Document can generate a large history:

```text
Small Document
      │
      ▼
Many edits
      │
      ▼
Large history
```

Likewise, a large Document can have a relatively small history if the user performs only a few edits.

This distinction becomes important when deciding how much history the editor should retain. Section 31.14 addresses that policy directly.

## Engineering Insight

There is no single number that describes the cost of undo/redo.

The relevant quantities include:

```text
Document size
      +
Changed-data size
      +
Number of history entries
      +
Allocation overhead
      +
Copying work
```

Snapshots make history cost strongly dependent on **Document size**.

Operation history makes it more dependent on **editing activity and changed-data size**.

That difference is the fundamental performance trade-off between the two approaches.

## Summary

Undo and redo add memory and processing costs whose character depends on the chosen strategy. Snapshots make history cost depend mainly on Document size, while operation history ties cost to the amount of changed data and editing activity; the relevant quantities include Document size, changed-data size, the number of history entries, allocation overhead, and copying work.

Now that those costs are understood, the next section examines how the editor can limit the amount of history it retains.

Continue to **Section 31.14, Limiting Undo History**.
