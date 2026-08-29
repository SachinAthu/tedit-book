# 26.4 Logical Text vs. Physical Layout

A gap buffer introduces an important distinction that does not exist in an ordinary contiguous string: **the order of characters in memory is no longer identical to their logical order in the document**.

Consider the following physical representation:

```text
H e l l o [ _ _ _ _ ] W o r l d
          └── gap ──┘
```

The characters are physically separated by unused storage. Yet the document itself is still:

```text
Hello World
```

The gap is not part of the document.

## The Logical Document

From the editor's perspective, the document is a continuous sequence:

```text
H e l l o W o r l d
```

The cursor might be positioned between `o` and `W`:

```text
H e l l o | W o r l d
          ^
        cursor
```

Nothing about the logical document indicates that unused memory exists at this position.

## The Physical Representation

Internally, however, the same document may be stored as:

```text
H e l l o [ _ _ _ _ ] W o r l d
          ^
       gap_start
```

The gap separates the two portions of the document:

```text
Text before gap
        │
        ▼
   +---------+
   |  Hello  |
   +---------+
        │
        ▼
   +---------+
   |   Gap   |
   +---------+
        │
        ▼
   +---------+
   |  World  |
   +---------+
        │
        ▼
Text after gap
```

The editor must therefore understand that the gap is **storage**, not text.

## Two Different Positions

This gives us two related concepts:

**Logical position**

The character position within the document as the user understands it.

```text
Hello|World
     ^
logical position = 5
```

**Physical position**

The index within the underlying array.

Because the gap occupies physical positions, these two positions are not always the same.

For example:

```text
Physical array:

Index:   0 1 2 3 4 5 6 7 8 9 10 11
         H e l l o _ _ _ W o  r  l
```

The logical character `W` appears immediately after `Hello`, but physically it occurs after the gap.

The gap buffer therefore needs a way to translate between the logical document and its physical storage.

## The Mapping Around the Gap

The mapping is straightforward on either side of the gap.

Characters before the gap can use their physical positions directly:

```text
Logical:   0 1 2 3 4
Physical:  0 1 2 3 4

            Hello
```

Characters after the gap are shifted physically by the size of the gap:

```text
Logical:    5  6  7  8  9
Physical:   9 10 11 12 13

             World
```

Conceptually:

```text
              gap
               ↓
Logical:  Hello|World
Physical: Hello[____]World
```

The gap is therefore invisible to the logical representation.

## Why This Distinction Matters

Without this distinction, it is easy to make incorrect assumptions about the buffer.

For example, code that simply iterates over every physical array position would encounter the gap:

```text
H e l l o _ _ _ _ W o r l d
          ↑
       not text
```

Those unused positions must not be rendered, counted as characters, or treated as document content.

A renderer or document operation must instead understand the logical sequence:

```text
H e l l o W o r l d
```

while the gap buffer itself manages the physical layout.

## The Gap as a Boundary

The most useful way to think about the structure is that the gap marks a boundary in the logical document:

```text
Logical document

       before cursor       after cursor
              │                  │
              ▼                  ▼
         +---------+          +---------+
         |  Hello  |          |  World  |
         +---------+          +---------+
              │                  │
              └────── cursor ────┘


Physical storage

         +---------+----------+---------+
         |  Hello  |   Gap    |  World  |
         +---------+----------+---------+
```

The gap sits exactly between the two logical portions.

This relationship is what allows the data structure to make insertion and deletion near the cursor efficient.

The next section examines the operation that maintains this relationship when the cursor moves: **moving the gap itself**.

## Summary

The gap buffer separates the logical document, a continuous sequence of characters, from the physical storage, in which the gap splits the same characters across both sides of an unused region. Logical and physical positions therefore differ when the gap has any size. Renderers and document operations must consult the logical sequence, while the gap buffers internal layout manages the physical placement.

In the next section, we will see how the buffer repositions the gap when the cursor moves, and what that movement costs.
