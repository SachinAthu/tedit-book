Review Chapter 26 — Arrays and Strings in:

Building a Terminal Text Editor — From First Principles to Professional Architecture

Determine whether the chapter's existing discussion of strings, characters, arrays, or text buffers creates any ambiguity concerning TAB characters.

Do NOT add TAB material merely for completeness.

Only modify the chapter if the current explanation could lead a reader to one of these incorrect conclusions:

- a TAB is equivalent to multiple spaces
- a TAB is a visual concept rather than a character/code point
- one text character necessarily equals one terminal display cell
- UTF-8 bytes are interchangeable with logical characters

## Current project representation

The Document stores Unicode code points in logical lines.

A literal TAB is:

U+0009

It is one logical code point.

Its UTF-8 representation is:

09

Its visual width is determined later by the Renderer.

## If the chapter needs clarification

Add only a concise clarification connecting arrays/strings to the project's chosen representation.

Do not introduce:

- tab stops
- tab width
- rendering
- cursor semantics
- configuration

Those belong elsewhere.

If the chapter is already correct and no ambiguity exists, make NO changes.

Report:

1. Whether a change is necessary.
2. The exact reason.
3. If necessary, the exact revised text.
