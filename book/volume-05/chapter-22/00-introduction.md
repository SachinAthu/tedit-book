# Volume V — File System

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
