# Volume V — File System

# Chapter 22 — Text Encoding

## Introduction

# Chapter 22 — Text Encoding

## Introduction

In Chapter 21, we examined how the editor communicates with the file system. We saw how a file is opened, how its contents are loaded into the in-memory Document, and how the complete Document is written back to disk when the user saves. The File Manager provides the boundary between the editor and the file system, while the Document remains the authoritative representation of the text being edited.

But there is an important question that file I/O alone cannot answer:

> **What do the bytes in a text file actually mean?**

A file is ultimately a sequence of bytes. Consider:

```text
48 65 6C 6C 6F
```

Those bytes can represent the text:

```text
Hello
```

when interpreted using an appropriate text encoding.

The distinction is easy to overlook because ASCII text makes bytes and characters appear almost interchangeable. For example:

```text
A → 41
B → 42
C → 43
```

Each character occupies exactly one byte. However, this relationship does not hold for all text.

Modern text can contain characters from many writing systems, symbols, and other Unicode code points. In UTF-8, a single Unicode code point may require multiple bytes:

```text
Character
    │
    ▼
Unicode Code Point
    │
    ▼
UTF-8 Encoding
    │
    ▼
One or more bytes
```

This creates an important distinction for a text editor:

```text
Byte ≠ Character
```

That distinction affects more than file loading and saving. It can influence how the Document stores text, how the Cursor identifies positions, how editing operations locate characters, and how the editor interprets the contents of a file.

There is another representation issue that becomes important when working with files created on different operating systems: **line endings**. A logical line break may be represented by `LF`:

```text
\n
```

or by `CRLF`:

```text
\r\n
```

Although both represent a transition from one line to another, they occupy different numbers of bytes in the file.

Therefore, text encoding is not merely a file-format detail. It is part of the boundary between the bytes stored on disk and the logical text represented by the editor.

In this chapter, we will examine that boundary carefully. We will distinguish bytes, characters, Unicode code points, and encodings; understand how UTF-8 represents Unicode text; examine the difference between byte positions and character positions; and see how LF and CRLF line endings affect the representation of a document.

The goal is not to build a complete Unicode text-processing system. Instead, we will establish the encoding model required by our editor and understand the engineering trade-offs that follow from that choice.

## Learning Objectives

Readers should be able to:

- distinguish bytes, characters, code points, and encodings
- explain the purpose of Unicode
- explain how UTF-8 represents Unicode code points
- explain why ASCII is a subset of UTF-8
- understand why a character may occupy multiple bytes
- distinguish LF from CRLF
- explain how line endings affect a text editor
- identify invalid UTF-8
- understand the encoding strategy chosen for our editor
- recognize the difference between byte positions and character positions

# 22.8 UTF-8 and the Text Buffer

This is where the chapter becomes specifically relevant to our editor.

Our Document stores text as a collection of lines, and ADR-013 deliberately chose the vector-of-lines representation for simplicity.

Now ask:

> If a line contains UTF-8, what exactly is stored?

Conceptually:

```text
Document
   │
   ├── Line 0 → UTF-8 bytes
   ├── Line 1 → UTF-8 bytes
   └── Line 2 → UTF-8 bytes
```

A line is ultimately stored as bytes, even though those bytes represent Unicode text.

This leads to an important distinction:

```text
Byte Offset
    vs
Character Position
```

For:

```text
hello
```

the two are equivalent.

For:

```text
café
```

they are not necessarily equivalent.

This section should connect carefully to the Cursor model established in Chapter 14 without reopening cursor movement design.

---

# 22.9 Byte Positions vs Character Positions

This deserves its own section because it is one of the most important consequences of UTF-8 for an editor.

Consider:

```text
café
```

The editor may store the line as UTF-8 bytes:

```text
63 61 66 C3 A9
```

The byte length is:

```text
5
```

while the number of Unicode code points is:

```text
4
```

Therefore:

```text
byte offset ≠ character position
```

Explain why this matters for:

- cursor movement
- insertion
- deletion
- status information
- rendering

But **do not redesign the Cursor here**. Chapter 14 already owns cursor movement. This chapter should establish the encoding constraint that later implementation must respect.

That separation is important for avoiding duplicate material.

---

# 22.10 Valid and Invalid UTF-8

Introduce the possibility that a file may contain bytes that do not form valid UTF-8.

Examples of possible situations:

```text
Valid UTF-8
     ↓
Decode normally
```

versus:

```text
Invalid byte sequence
     ↓
What should the editor do?
```

Discuss possible policies:

1. Reject the file
2. Replace invalid sequences
3. Treat the document as arbitrary bytes
4. Preserve invalid bytes while displaying them specially

Then establish the project's baseline policy.

Because the existing SRS only says that the editor operates on plain text files "encoded in a supported character encoding," this is a design decision that should be explicitly clarified rather than silently assumed.

---

# 22.11 Line Endings

Now move from **character encoding** to the other major text-representation issue in the outline: line endings.

Explain that a logical line break can be represented differently:

```text
Unix / Linux / macOS
LF
\n
```

and:

```text
Windows
CRLF
\r\n
```

Show:

```text
Hello\nWorld
```

versus:

```text
Hello\r\nWorld
```

The important distinction:

> A line ending is part of the file representation, not normally part of the logical text line stored by the editor.

This connects naturally to the vector-of-lines Document model.

---

# 22.12 LF vs CRLF

Give a focused comparison.

| Representation | Bytes   | Common environment |
| -------------- | ------- | ------------------ |
| LF             | `0A`    | Unix/Linux         |
| CRLF           | `0D 0A` | Windows            |

Then explain why this matters during loading:

```text
File
Hello\r\n
World\r\n
      │
      ▼
Document
"Hello"
"World"
```

The `\r` should not accidentally become part of the document's visible text.

---

# 22.13 Line Ending Policy

This is where we make the project's architectural choice.

Questions to answer:

- What does the Document store?
- What happens when CRLF is loaded?
- What line ending is written during save?
- Should the original line-ending style be preserved?
- What happens with mixed line endings?

For the baseline editor, I recommend:

### Internal representation

Store logical lines without their line-ending bytes.

### Loading

Recognize both:

```text
LF
CRLF
```

and convert them into the same internal line representation.

### Saving

Use a single canonical line-ending policy.

However, **we should decide whether the baseline policy is always LF or whether the editor preserves the detected style**.

For the educational architecture, I recommend **canonical LF internally and on save** because it is simpler and consistent with the Linux-primary project.

But we should explicitly discuss the trade-off: saving a Windows CRLF file would normalize it to LF.

If we want compatibility with existing files to be more user-friendly, preservation could instead be chosen. That decision should be recorded as an ADR rather than buried in the chapter.

---

# 22.14 Encoding and Saving

This section should connect Chapter 22 back to Chapter 21 without repeating its file-system material.

Chapter 21 answers:

> How do we read and write the file?

Chapter 22 answers:

> What bytes should those operations interpret and produce?

The complete conceptual pipeline becomes:

```text
                 LOAD

File
 │
 │ bytes
 ▼
Decode UTF-8
 │
 ▼
Document
 │
 │ editing
 ▼
Document
 │
 │ encode UTF-8
 ▼
File
```

This distinction between **I/O mechanism** and **text representation** is an important architectural boundary.

---

# 22.15 Encoding Strategy for Our Editor

This should be the chapter's project-specific design section.

Proposed baseline:

```text
Encoding:
UTF-8

Internal text:
UTF-8 byte sequences

Supported line endings:
LF
CRLF

Internal line representation:
Logical lines without line-ending bytes

Output:
UTF-8 with LF line endings
```

Then explain the consequences.

### Advantages

- Simple
- Native to Linux
- ASCII compatible
- Supports international text
- No conversion to another internal character encoding
- Easy to preserve UTF-8 data

### Limitations

- Character positions are not necessarily byte positions
- Unicode-aware cursor movement requires decoding
- Full grapheme handling is not provided
- Invalid UTF-8 requires an explicit policy
- CRLF files may be normalized

This is also a good location to introduce a new ADR, likely **ADR-021 — Text Encoding Strategy**.

---

# 22.16 Complexity and Trade-offs

This should be relatively focused.

Discuss:

### Memory

UTF-8 uses a variable number of bytes per code point.

Therefore:

```text
Memory ≈ number of UTF-8 bytes
```

not simply:

```text
number of characters
```

### Processing

ASCII-heavy text is cheap to process because each character is one byte.

Unicode text may require scanning multiple bytes to locate character boundaries.

### Trade-off

Keeping UTF-8 directly in the Document avoids a conversion layer, but higher-level operations must understand UTF-8 boundaries when they operate on characters rather than raw bytes.

Do **not** repeat the full complexity analysis of the vector-of-lines structure from Chapter 13 or file loading/saving from Chapter 21.

---

# 22.18 Engineering Insight — Text Is a Layered Representation

End the conceptual portion with a broader model:

```text
             User sees
                 │
                 ▼
          Visual Characters
                 │
                 ▼
          Unicode Code Points
                 │
                 ▼
             UTF-8 Bytes
                 │
                 ▼
            File System
```

A text editor operates across all of these layers.

A bug can occur whenever the program assumes that two layers are interchangeable.

For example:

```text
"character" == "byte"
```

is true for ASCII but false in general.

That is precisely why encoding must be treated as an architectural concern rather than a minor file-format detail.

---

One other point: **I would add an ADR-021 for the editor's text-encoding policy**, because the current ADR set reaches ADR-020 with the full-document persistence model, and the encoding policy materially affects Document, Cursor, and File Manager behavior. The existing project constitution explicitly says major design decisions should be recorded as ADRs.
