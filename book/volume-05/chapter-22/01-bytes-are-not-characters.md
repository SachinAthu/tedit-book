# 22.1 Bytes Are Not Characters

A text file is ultimately stored as a sequence of **bytes**.

For example, a file containing:

```text
Hello
```

can be represented as:

```text
48 65 6C 6C 6F
```

Each hexadecimal value represents one byte.

At this level, the file system does not know that these bytes represent the word `Hello`. It simply stores the bytes. Their meaning comes from how the application interprets them.

For a text editor, this distinction is fundamental:

```text
File
  │
  │ bytes
  ▼
Text Encoding
  │
  │ interpretation
  ▼
Characters
```

The encoding defines how sequences of bytes correspond to text.

## ASCII Makes the Distinction Easy to Miss

ASCII provides a particularly simple example because each ASCII character has a one-byte representation.

For example:

```text
A → 0x41
B → 0x42
C → 0x43
```

Therefore:

```text
ABC
```

can be stored as:

```text
41 42 43
```

There are three bytes and three characters.

This can lead to an assumption that is convenient but incorrect:

```text
one byte = one character
```

That relationship is only true for encodings and characters where a character occupies exactly one byte.

## UTF-8 Changes the Relationship

UTF-8 is a variable-width encoding. A Unicode code point may require more than one byte when represented in UTF-8.

For example:

```text
A
```

is represented by one byte:

```text
41
```

while some other characters require multiple bytes.

Conceptually:

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
Multiple bytes
```

Therefore, a text such as:

```text
café
```

cannot necessarily be understood simply by counting its bytes.

The important principle is:

> **The number of bytes in a text sequence is not necessarily the number of characters it represents.**

This distinction becomes important whenever the editor needs to locate, insert, delete, or count characters.

## Bytes Are What the File Contains

When the File Manager loads a document, it initially receives bytes from the file system.

Conceptually:

```text
Disk
 │
 │ bytes
 ▼
File Manager
 │
 ▼
Document
```

The Document, however, represents **text**, not merely an arbitrary collection of unrelated bytes.

The editor therefore needs an encoding convention that defines how those bytes are interpreted.

For our editor, that encoding will be **UTF-8**.

This gives us a useful conceptual separation:

```text
Physical representation
        │
        ▼
   UTF-8 bytes
        │
        ▼
Logical representation
        │
        ▼
      Text
```

The bytes remain the underlying storage representation, while the characters they encode are the logical text the editor operates on.

## Why This Matters for an Editor

Suppose an editor assumes that every character occupies one byte.

For ASCII text, that assumption appears to work:

```text
Hello
```

can be traversed one byte at a time.

But with UTF-8 text, advancing by one byte does not necessarily mean advancing by one character.

An operation that incorrectly assumes:

```text
next character = next byte
```

can enter the middle of a multi-byte UTF-8 sequence.

That can lead to corrupted text, incorrect cursor positions, or invalid UTF-8.

This is why the distinction established here will matter later when we discuss the relationship between **byte offsets and character positions**.

For now, the essential idea is simple:

```text
Bytes
  ≠
Characters
```

Bytes are the physical representation stored in the file. Characters are part of the logical text represented by those bytes. An encoding such as UTF-8 defines the relationship between the two.
