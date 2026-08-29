# 26.9 Representing Multiple Lines

So far, we have considered a gap buffer as though the document were one continuous sequence of characters:

```text
abc[____]def
```

Our editor, however, does not treat a document simply as one long character sequence. The current design represents a document as a collection of lines, with each line independently stored in memory.

This raises an important design question:

> Where does the gap buffer fit when a document contains multiple lines?

There are several possible ways to organize it.

## One Gap Buffer for the Entire Document

The first approach is to treat the entire document as one character sequence.

For example:

```text
Hello
World
Editor
```

could be represented conceptually as:

```text
Hello\nWorld\nEditor
```

and the gap buffer could look like:

```text
Hello
[____]
World
Editor
```

The newline characters are simply part of the sequence.

The advantage is that there is only one gap:

```text
Document
    │
    ▼
+----------------------+
|      Gap Buffer      |
+----------------------+
```

The entire document therefore has one continuous physical representation.

However, operations involving lines now need to account for newline characters explicitly. Splitting a line, joining two lines, and locating a particular row require working within that single character sequence.

## One Gap Buffer per Line

A different approach is to keep the document's line structure and give every line its own gap buffer:

```text
Document
   │
   ├── Line 0 → Gap Buffer
   ├── Line 1 → Gap Buffer
   └── Line 2 → Gap Buffer
```

For example:

```text
Line 0: Hello[____]
Line 1: World[____]
Line 2: Editor[____]
```

Now each line has an independent editing workspace.

If the cursor is on `Line 1`, its gap can remain positioned around the cursor while the other lines retain their own gaps.

This fits naturally with the existing row-and-column model:

```text
Cursor
   │
   ├── row
   └── column
```

The row selects the line, while the column identifies the logical position within that line.

## The Two Approaches

The difference can be summarized as:

```text
One gap buffer:

Document
    │
    ▼
[ entire character sequence ]
              ^
             gap
```

versus:

```text
Multiple gap buffers:

Document
    │
    ├── [Line 0][gap]
    ├── [Line 1][gap]
    └── [Line 2][gap]
```

Neither arrangement is automatically correct for every editor.

They represent different ways of combining a **text organization model** with a **storage strategy**.

## What Happens When a Line Is Split?

Consider:

```text
HelloWorld
     ^
   cursor
```

Pressing Enter produces:

```text
Hello
World
```

With a per-line representation, the operation naturally becomes:

```text
One Gap Buffer
      │
      ▼
Split into two buffers

Hello[...]
World[...]
```

The document gains another line.

With a single document-wide gap buffer, the newline becomes part of the same underlying sequence:

```text
Hello\nWorld
```

The logical line structure is then interpreted from the newline boundaries.

This demonstrates an important architectural distinction:

> The way a document is logically organized does not necessarily dictate how its characters must be physically stored.

## Why This Matters to Our Editor

Our current architecture deliberately separates the **Document** from its internal representation.

The Document is responsible for the editable text, while the specific representation is an implementation decision. ADR-013 selected a vector of lines because it maps naturally to the editor's row/column model and prioritizes simplicity.

The gap buffer is therefore useful here primarily as a **data-structure alternative**.

We do not need to redesign the Document architecture simply because we are studying a different representation.

Instead, we can ask:

```text
Document abstraction
        │
        ▼
Which representation
best satisfies its needs?
        │
        ├── Vector of lines
        ├── Gap buffer
        ├── Piece table
        └── Rope
```

This separation between **what the Document provides** and **how the text is stored** is an important software-engineering principle.

## Summary

A gap buffer can be organized either as one buffer over the entire document with newline characters in the sequence, or as a separate gap buffer per line. The editor's Document abstraction separates logical organization (a collection of lines) from physical storage, so the gap buffer remains a data-structure alternative rather than a redesign of the architecture.

In the next section, we will examine why the gap buffer's performance depends on where editing actually happens: the locality of editing and cursor movement.
