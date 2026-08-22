We are editing Chapter 33 — Configuration of:

Building a Terminal Text Editor — From First Principles to Professional Architecture

Review the chapter and add the configuration concepts needed for TAB behavior.

Do not redesign the configuration architecture.

## Existing architectural decisions

A TAB is:

U+0009

inside the Document.

The Renderer determines its visual width.

The Cursor operates on logical positions.

Therefore tab width is a rendering configuration value, not a Document representation value.

## Add tab width

Introduce a configuration value such as:

tab_width

Its purpose is:

> Number of visual terminal columns between tab stops.

Use a sensible baseline default consistent with the rest of the project.

If the existing project configuration already specifies a value, preserve it.

Do not invent a conflicting default.

Explain that:

tab_width = 4

means tab stops occur at:

0, 4, 8, 12, ...

The configuration affects rendering, not the stored Document.

## If the project supports insertion policy

If the existing architecture includes an indentation/insertion configuration, distinguish:

1. Literal TAB insertion
2. Spaces-for-tabs insertion policy
3. Visual tab width

These are NOT the same thing.

For example:

tab_width = 4

does NOT mean:

"replace every TAB with four spaces."

A file can contain literal U+0009 while being rendered with a width of 4.

Only add "insert spaces instead of TAB" if the existing configuration architecture has a natural place for it.

Do not invent a large indentation system if the book has not already established one.

## Configuration boundaries

Make clear:

Configuration
│
├── tab_width
│
▼
Renderer

The Document does not change when tab_width changes.

Example:

Document:

A<TAB>B

With tab_width = 4:

A···B

With tab_width = 8:

A·······B

Same Document.
Different visual representation.

## Avoid duplication

Do not explain UTF-8 encoding.
Chapter 22 does that.

Do not explain cursor movement.
Chapter 14 does that.

Do not explain tab-stop rendering algorithms in depth.
Chapter 19 does that.

This chapter should focus on:

- what configuration exists
- what the setting means
- which subsystem consumes it
- why it should not be stored in the Document

After editing, provide the exact revised sections and identify any assumptions taken from existing project configuration.
