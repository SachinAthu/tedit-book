# Chapter Summary

Summarize:

- Files contain bytes.
- Encodings give those bytes meaning.
- Unicode defines code points; UTF-8 encodes them as bytes.
- ASCII is compatible with UTF-8.
- UTF-8 is variable-width.
- Byte offsets and character positions can differ.
- Line endings are another representation-level concern.
- LF and CRLF represent line boundaries differently.
- The editor should separate logical lines from physical line-ending bytes.
- The project's encoding policy should be explicit.

## Exercises

### Review Questions

1. What is the difference between Unicode and UTF-8?
2. Why can a UTF-8 character occupy multiple bytes?
3. Why is ASCII compatible with UTF-8?
4. Why can byte offsets not always be used as character positions?
5. What is the difference between LF and CRLF?

### Thought Experiments

- What happens if an editor deletes one byte from the middle of a multi-byte UTF-8 sequence?
- What happens if a CRLF file is loaded without removing `\r`?
- Why would treating every byte as a character work for English text but fail for international text?

### Optional Challenge

Design a function that walks through a UTF-8 line and identifies the beginning of each code point without converting the entire line into another representation.

## Common Mistakes

A concise section covering mistakes such as:

- assuming one character equals one byte
- confusing Unicode with UTF-8
- treating `\r\n` as two logical line breaks
- counting UTF-8 bytes as characters
- splitting a UTF-8 sequence in the middle
- assuming every file is valid UTF-8
- silently mixing byte offsets and character positions

These mistakes are especially valuable because they expose bugs that can otherwise look completely mysterious.

## Preview of the Next Chapter

Chapter 22 establishes **what the bytes mean**.

Chapter 23 should establish **how to safely modify those bytes on disk**.

The next chapter can therefore focus entirely on:

- temporary files
- atomic saves
- partial writes
- interrupted saves
- crash scenarios
- recovery strategies
- preserving the user's original file

This keeps Chapter 23 clearly separate from Chapter 21's basic file I/O and Chapter 22's encoding concerns.
