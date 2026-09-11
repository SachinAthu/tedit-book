# 32.9 Syntax Highlighting and the Text Buffer

Syntax highlighting analyzes the contents of the Document, but it must coexist with the editor's text-buffer architecture.

This raises an important design question:

> Should highlighting information be stored inside the text buffer alongside the text?

The answer for this editor is **no**.

The text buffer remains responsible for storing and modifying source text. Syntax highlighting is derived information that can be generated from that text.

This distinction prevents language-specific concerns from leaking into the core document representation.

## The Text Buffer Has One Primary Responsibility

The Document represents the actual source text.

Conceptually:

```text
Document
│
├── Line 0
├── Line 1
├── Line 2
├── ...
└── Line N
```

Editing operations modify this structure:

```text
Insert
Delete
Split line
Join lines
Replace text
```

The Document does not need to know whether its contents are:

```text
C
Python
JavaScript
JSON
Markdown
plain text
```

It simply stores text.

Syntax highlighting is layered on top:

```text
              Document
                  │
                  │ text
                  ▼
          Syntax Highlighting
                  │
                  │ derived information
                  ▼
               Renderer
```

This allows the same text-buffer implementation to support many languages.

## Why Not Store Highlighting With Each Character?

A tempting design is to extend every character or code point in the Document with a highlight class:

```text
Code Point
├── value
└── highlight class
```

This appears convenient because the syntax information is immediately available alongside the text.

However, it creates several problems.

First, the core Document becomes language-aware.

Second, every edit must potentially update both text and syntax metadata.

Third, syntax information becomes tightly coupled to the lifetime and internal representation of the text buffer.

Fourth, changing the syntax-highlighting implementation becomes harder because the buffer now depends on it.

The alternative is:

```text
Document
    │
    └── source text

Syntax State
    │
    └── derived highlight information
```

The two structures can evolve independently.

## Derived State Versus Authoritative State

The distinction can be summarized as:

```text
Authoritative state
        │
        ▼
     Document
        │
        │ analyzed
        ▼
   Derived state
        │
        ├── Tokens
        ├── Syntax state
        └── Highlight ranges
```

If the derived state is lost:

```text
Highlight information lost
        │
        ▼
Re-analyze Document
        │
        ▼
Highlight information restored
```

If the Document is lost, however, the actual source text is gone.

This difference establishes ownership.

The Document is fundamental.

Syntax information is disposable and recomputable.

## The Cost of Keeping Syntax Data in the Buffer

Suppose a Document contains one million code points.

If every code point also stores syntax metadata, the memory cost of the Document increases substantially.

Even a small metadata field multiplied across a large document can become significant.

More importantly, much of that information is unnecessary.

A large region of ordinary source text may use the same default class:

```text
NORMAL
```

Storing an explicit class for every character wastes space.

A range-based representation can instead describe only meaningful changes:

```text
Range 1 → KEYWORD
Range 2 → IDENTIFIER
Range 3 → NUMBER
```

Unclassified regions can implicitly use the default class.

This makes the syntax layer independent of the size and internal representation of individual Document elements.

## The Text Buffer Should Not Become a Syntax Cache

Another tempting design is to make the Document responsible for caching syntax information:

```text
Document
├── text
├── cursor
├── syntax cache
└── language state
```

This is undesirable because the Document then becomes responsible for information that does not define the document itself.

A cleaner architecture is:

```text
Document
└── source text

Syntax Subsystem
├── language
├── tokens
├── syntax state
└── highlight information
```

The syntax subsystem can maintain its own derived state.

The Document remains language-neutral.

## Edits Create Invalidation

The separation becomes especially useful when the user edits the document.

Suppose the source initially contains:

```text
/* configuration
   values */
int count = 10;
```

The syntax subsystem knows that the first two lines are part of a comment.

Now the user deletes the opening:

```text
configuration
   values */
int count = 10;
```

The syntax interpretation has changed dramatically.

The Document has been modified through the normal editing path.

The syntax subsystem must now reconsider its derived state.

Conceptually:

```text
Document edit
     │
     ▼
Document changes
     │
     ▼
Existing syntax data may be stale
     │
     ▼
Re-analysis
     │
     ▼
Updated syntax data
```

The important point is that the edit does not directly manipulate syntax ranges.

The syntax subsystem reacts to the changed source text.

## Why This Works Well With Undo and Redo

Undo and redo operate on document-editing operations.

Syntax information does not need to become part of the undo history.

For example:

```text
User inserts text
       │
       ├── Document changes
       │
       └── Undo history records edit
```

Syntax highlighting then responds:

```text
Document changes
       │
       ▼
Syntax invalidation
       │
       ▼
Re-analysis
```

When the user performs undo:

```text
Undo
  │
  ▼
Document restored
  │
  ▼
Syntax re-evaluated
```

This keeps the responsibilities established by the undo/redo architecture intact.

Syntax metadata is not user-authored document state, so it does not belong in the operation history.

## Why This Works With Search

The same principle applies to search.

Search operates on the Document's text.

Syntax highlighting operates on the Document's text.

Neither needs to own or modify the other.

Conceptually:

```text
              Document
             /        \
            /          \
           ▼            ▼
       Search         Syntax
           │            │
           ▼            ▼
     Search State   Highlight State
```

Both are derived systems that consume the authoritative document.

This avoids unnecessary coupling between professional editor features.

## Syntax Information and Line Changes

The line-oriented nature of the Document makes syntax invalidation particularly interesting.

A change can affect:

```text
one character
```

but its syntactic consequences may extend across:

```text
one line
many lines
the remainder of the document
```

For example, inserting:

```text
/*
```

at the beginning of a line may cause every subsequent line to be interpreted as comment text until a closing delimiter is encountered.

Therefore, the syntax subsystem cannot always assume:

```text
changed line
    ↓
only changed line needs re-analysis
```

Instead:

```text
changed line
    ↓
re-analyze
    ↓
syntax state changes?
    │
    ├── no → stop
    │
    └── yes
          ↓
      analyze next line
          ↓
      continue until stable
```

This provides a useful path toward incremental highlighting without changing the Document's representation.

## Syntax State at Line Boundaries

To support this approach, the syntax subsystem can conceptually associate state with each analyzed line.

For example:

```text
Line 0
Input:  NORMAL
Output: IN_COMMENT

Line 1
Input:  IN_COMMENT
Output: IN_COMMENT

Line 2
Input:  IN_COMMENT
Output: NORMAL
```

The Document itself does not need to store these states.

They belong to the syntax subsystem.

This distinction is important because the same line of text can have different highlighting depending on the syntax state inherited from previous lines.

For example:

```text
/* text */
```

may be ordinary comment text when analyzed independently, but a syntax-aware analyzer must understand whether the opening or closing delimiter belongs to a larger construct.

## Stable Boundaries

Incremental highlighting becomes easier when the syntax state eventually returns to a previously known state.

Suppose an edit occurs on line 10:

```text
Line 10 changed
      │
      ▼
Re-analyze line 10
      │
      ▼
Line 11 changes?
      │
      ├── no
      │
      └── yes
            │
            ▼
      continue forward
```

If line 15 produces the same outgoing syntax state it produced before the edit, later lines may no longer need to be reconsidered.

Conceptually:

```text
Edit
 │
 ▼
10 → 11 → 12 → 13 → 14 → 15
                         │
                         └── state stable
                                │
                                ▼
                          stop re-analysis
```

This is a practical optimization for an interactive editor.

It avoids repeatedly analyzing the entire Document after every small edit.

## Highlighting and Document Lifetime

The syntax subsystem must also respect the lifetime of the Document.

If a Document is closed:

```text
Document destroyed
```

its associated syntax information must no longer be used.

Similarly, if a different Document becomes active:

```text
Document A
   │
   └── Syntax state A

Document B
   │
   └── Syntax state B
```

the syntax state associated with Document A must not accidentally be applied to Document B.

This reinforces the idea that syntax state belongs to the analyzed Document state rather than to the renderer globally.

## Avoiding Stale References

If tokens or highlight ranges refer directly into internal Document storage, edits can make those references invalid.

For example:

```text
Token
└── pointer → document data
```

After an operation that reallocates or restructures the underlying storage, the pointer might no longer refer to the intended text.

A safer conceptual model is for syntax information to refer to stable logical positions or ranges:

```text
Token
├── document position
├── range
└── token type
```

The syntax subsystem can then resolve those positions against the current Document state.

The exact memory-management strategy depends on the underlying implementation, but the architectural rule remains:

> **Derived syntax data must not accidentally impose unsafe ownership or lifetime requirements on the Document.**

## Multiple Syntax Representations

The syntax subsystem may maintain several layers of derived information:

```text
Document
    │
    ▼
Tokens
    │
    ▼
Syntax interpretation
    │
    ▼
Highlight ranges
```

These layers do not all have to exist permanently.

A simple implementation might generate tokens and immediately produce highlight ranges.

A more sophisticated implementation might retain tokens because later analysis needs them.

The important point is that each representation remains disposable.

For example:

```text
Document
   │
   ├── authoritative
   │
   ▼
Tokens
   │
   └── disposable
        │
        ▼
Highlight ranges
   │
   └── disposable
```

This makes the system easier to invalidate and rebuild.

## The Buffer Does Not Need to Know the Language

One of the strongest benefits of this design is language independence.

The same Document implementation can contain:

```text
C source
Python source
Markdown
JSON
plain text
```

The Document does not need to change.

Only the syntax subsystem changes its analysis rules:

```text
Document
   │
   ├── C language definition
   │
   ├── Python language definition
   │
   ├── Markdown language definition
   │
   └── Plain-text mode
```

This keeps the core text model stable while allowing language-specific behavior to remain isolated.

## A Clean Architectural Relationship

The resulting relationship between the text buffer and syntax subsystem is:

```text
                 ┌──────────────┐
                 │   Document   │
                 │              │
                 │ source text  │
                 └──────┬───────┘
                        │
                        │ read
                        ▼
                 ┌──────────────┐
                 │    Syntax    │
                 │  Subsystem   │
                 │              │
                 │ lexer        │
                 │ tokens       │
                 │ parser       │
                 │ highlights   │
                 └──────┬───────┘
                        │
                        │ semantic classes
                        ▼
                 ┌──────────────┐
                 │    Theme     │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │   Renderer   │
                 └──────────────┘
```

The direction of dependency is important.

The syntax subsystem depends on the Document's contents.

The Document does not depend on syntax highlighting.

This preserves the architecture's low coupling and keeps the core editing model independent of professional presentation features.

## The Key Design Principle

Syntax highlighting should therefore be treated as a **derived view of the text buffer**.

The relationship can be summarized as:

```text
Document
   │
   │ authoritative
   ▼
Source Text
   │
   │ analyzed
   ▼
Syntax Information
   │
   │ interpreted
   ▼
Highlight Classes
   │
   │ styled
   ▼
Rendered Text
```

The source text is stable and authoritative.

The syntax information can be recalculated.

The visual appearance can change independently.

This three-layer separation gives the editor a strong foundation for both simple highlighting and more advanced syntax infrastructure later.

## Summary

Syntax highlighting is a derived view of the text buffer: the Document remains the single authoritative source of text, while the syntax subsystem reads it and produces tokens and highlight ranges that are disposable and can be recalculated. Derived syntax data must never impose unsafe ownership or lifetime requirements on the Document, and the two can remain decoupled—the syntax subsystem depends on the Document's contents, but the Document does not depend on syntax highlighting. Because of this separation, the same Document implementation works for many languages, and adding a language definition is enough to change behavior without touching the core text model.

The next concern is the final presentation layer: **themes and visual styles**. A semantic class such as `KEYWORD` still needs to become an actual terminal style before the renderer can display it. That is the subject of **Section 32.10, Themes and Visual Styles**.
