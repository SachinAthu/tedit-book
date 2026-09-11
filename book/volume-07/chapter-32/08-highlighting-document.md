# 32.8 Highlighting the Document

The syntax subsystem can now produce tokens and highlight classes, but the editor still has to answer a practical question:

> How does highlighting information become associated with the text currently stored in the Document?

The answer should preserve the architecture established throughout the editor.

The Document remains responsible for source text.

The syntax subsystem derives highlighting information from that text.

The Renderer consumes the resulting information when constructing the visible frame.

The relationship is therefore:

```text
Document
    │
    │ read
    ▼
Syntax Subsystem
    │
    │ produces
    ▼
Highlight Information
    │
    │ consumed by
    ▼
Renderer
```

The highlighting information does not become part of the authoritative text representation.

## The Document Remains the Source of Truth

Suppose the Document contains:

```text
int count = 42;
```

The syntax subsystem may determine:

```text
int       → KEYWORD
count     → IDENTIFIER
=         → OPERATOR
42        → NUMBER
;         → PUNCTUATION
```

The Document itself remains unchanged:

```text
int count = 42;
```

There is no need to transform it into something like:

```text
<keyword>int</keyword>
<identifier>count</identifier>
<operator>=</operator>
<number>42</number>
```

Such a representation would mix content with presentation metadata and make the text buffer dependent on language-specific syntax information.

Instead, the editor keeps two separate concepts:

```text
Authoritative data:

Document
└── int count = 42;


Derived data:

Highlight information
├── KEYWORD
├── IDENTIFIER
├── OPERATOR
├── NUMBER
└── PUNCTUATION
```

This separation is fundamental.

## Highlight Ranges Describe Document Regions

A convenient conceptual representation is a collection of highlight ranges.

Each range identifies:

```text
Highlight Range
├── start
├── end
└── highlight class
```

For example:

```text
int count = 42;
```

could produce:

```text
(0, 0) → (0, 3)   KEYWORD
(0, 4) → (0, 9)   IDENTIFIER
(0,10) → (0,11)   OPERATOR
(0,12) → (0,14)   NUMBER
(0,14) → (0,15)   PUNCTUATION
```

Here `count` is shown as `IDENTIFIER`, its raw token classification; a later syntax-analysis stage may refine it to `VARIABLE`, `FUNCTION`, or `TYPE` as discussed in Section 32.5.

These ranges are expressed using logical Document coordinates.

They are not screen coordinates.

That distinction matters because the same document can be displayed at many different viewport positions.

## Logical Coordinates Versus Screen Coordinates

Suppose the document contains:

```text
10: int count = 42;
11: return count;
12: ...
```

The syntax subsystem might identify a token at:

```text
Document position:
(row 11, column 0)
```

The user might currently have the viewport positioned so that row 11 appears on screen row 3.

The syntax information does not change.

```text
Document coordinate
        │
        ▼
(row 11, column 0)
        │
        │ viewport transformation
        ▼
Screen coordinate
(row 3, column ...)
```

The transformation belongs to the rendering process.

Syntax information should therefore remain independent of scrolling.

This allows syntax analysis to be performed without knowing anything about the current terminal dimensions or viewport position.

## Highlighting a Line

A simple implementation can think about highlighting one document line at a time.

Consider:

```text
const int limit = 100;
```

The syntax subsystem could produce:

```text
const       → KEYWORD
int         → KEYWORD
limit       → IDENTIFIER
=           → OPERATOR
100         → NUMBER
;           → PUNCTUATION
```

Conceptually, the line becomes a sequence of classified regions:

```text
┌─────────┬──────┬───────┬───┬─────┬───┐
│ KEYWORD │ TYPE │ IDENT │ = │ NUM │ ; │
└─────────┴──────┴───────┴───┴─────┴───┘
```

The source characters have not changed.

Only their derived classification has changed.

The renderer can later walk through the visible characters and determine which highlight class applies to each region.

## Highlighting the Visible Portion

The syntax subsystem may know about the entire Document:

```text
Document
├── line 0
├── line 1
├── line 2
├── ...
├── line 500
└── line 501
```

The renderer, however, may only display:

```text
line 200
line 201
line 202
...
line 230
```

The renderer does not need to render syntax information for invisible lines.

Conceptually:

```text
                    Document
                       │
             ┌─────────┴─────────┐
             │                   │
       syntax information     viewport
             │                   │
             └─────────┬─────────┘
                       ▼
                visible regions
                       │
                       ▼
                    Renderer
```

This separation is important for performance.

Syntax information can be associated with the logical document, while rendering only consumes the portion relevant to the current viewport.

## Applying Classes During Rendering

The Renderer already has the responsibility of constructing the visible frame.

Syntax highlighting does not change that responsibility.

Instead, the renderer can conceptually perform an additional lookup while processing each visible region:

```text
Document text
      │
      ├── character/range
      │
      ▼
Highlight information
      │
      ▼
Highlight class
      │
      ▼
Theme
      │
      ▼
Visual attributes
      │
      ▼
Screen buffer
```

For example:

```text
Text:
int

Class:
KEYWORD

Theme:
KEYWORD → visual style

Renderer:
apply that style to the cells representing "int"
```

The important point is that the renderer does not determine that `int` is a keyword.

That decision has already been made by the syntax subsystem.

The renderer merely uses the classification.

## Highlighting and the Screen Buffer

The editor's screen buffer represents the frame that will eventually be written to the terminal.

Syntax highlighting contributes visual attributes to the cells representing document text.

Conceptually:

```text
Document
   │
   ▼
Syntax information
   │
   ▼
Highlight class
   │
   ▼
Theme style
   │
   ▼
Screen-cell attributes
   │
   ▼
Screen buffer
   │
   ▼
Terminal
```

This does not require syntax information to be stored inside the screen buffer permanently.

The screen buffer remains a representation of the current rendered frame.

If the user scrolls, types, or resizes the terminal, the next frame can be constructed using the same document and the appropriate syntax information.

## Whitespace and Unhighlighted Text

Not every character requires an explicit highlight range.

For example:

```text
int count = 42;
```

contains spaces that may simply use the default visual class.

A conceptual representation might therefore be:

```text
KEYWORD
    int

NORMAL
       [space]

IDENTIFIER
        count

NORMAL
             [space]

OPERATOR
             =

NORMAL
              [space]

NUMBER
               42

PUNCTUATION
                 ;
```

An implementation does not necessarily need to create a separate `NORMAL` range for every unclassified character.

It can instead treat the absence of a more specific range as the default class.

This can reduce unnecessary derived data.

## Overlapping Ranges

Highlight ranges can become more complicated when multiple analysis rules apply to the same region.

Consider:

```text
"return"
```

The characters `return` resemble a keyword, but they are inside a string.

The syntax subsystem must ensure that the effective result is:

```text
STRING
```

rather than:

```text
KEYWORD
```

This means the system needs a deterministic way to resolve overlapping or competing classifications.

One conceptual approach is to process more specific or higher-priority regions first.

Another is to have the parser produce a final classification rather than several competing classifications.

The exact implementation is a design choice.

The architectural requirement is simpler:

> **For every rendered document region, the syntax subsystem must provide a deterministic effective highlight class.**

## Highlighting Does Not Change Cursor Behavior

The Cursor remains independent of syntax highlighting.

Suppose the cursor is positioned inside:

```text
int count = 42;
```

The syntax subsystem may classify `count` as an identifier.

The Cursor still stores its logical document position.

```text
Cursor
└── row
    column
    preferred column

Syntax
└── highlight ranges
```

The two pieces of state may interact during rendering, but neither owns the other.

The renderer combines them when producing the final visual representation.

For example:

```text
Syntax style
     +
Cursor style
     ↓
Rendered cell
```

This is another reason not to store terminal formatting directly inside tokens.

The final appearance of a cell can depend on several independent sources of visual state.

## Highlighting Does Not Change Editing

Similarly, syntax information must not interfere with editing operations.

An insertion:

```text
insert character
```

still modifies the Document through the normal editing machinery.

The syntax subsystem does not directly modify the text.

Instead:

```text
User input
    │
    ▼
Command / editing operation
    │
    ▼
Document changes
    │
    ▼
Syntax information becomes stale
    │
    ▼
Syntax analysis
    │
    ▼
Updated highlighting
```

This maintains the established separation between editing state and derived analysis.

It also means that syntax highlighting can be disabled without changing the underlying editing model.

## Whole-Document Highlighting

The simplest syntax implementation can analyze the entire Document whenever highlighting needs to be regenerated.

Conceptually:

```text
Document
   │
   ▼
Lex entire document
   │
   ▼
Analyze entire document
   │
   ▼
Generate highlight information
```

This approach is straightforward and easy to reason about.

For a small document, it may be perfectly adequate.

Its main disadvantage is that a tiny edit can cause a large amount of unnecessary work.

For example:

```text
line 20 changes
```

could cause:

```text
lines 0–500
     ↓
re-analyzed
```

even though most of the document was unaffected.

This is one of the first places where the derived-state model exposes a performance question.

## Per-Line Highlighting

A more efficient approach is to associate highlighting information with individual lines.

For example:

```text
Document
├── line 0  → highlight state
├── line 1  → highlight state
├── line 2  → highlight state
├── ...
└── line N  → highlight state
```

When a line changes, the editor can reconsider that line rather than automatically processing the entire document.

However, this introduces an important complication.

Syntax state can cross line boundaries.

For example:

```text
/* start of comment
   still inside comment
   end of comment */
```

The second line cannot always be analyzed independently.

The syntax state at the beginning of a line may depend on what happened on previous lines.

Therefore, line-level highlighting needs some representation of lexical state.

## Propagating Syntax State

Consider:

```text
/*
   explanation
   more explanation
*/
int value = 10;
```

The lexer enters comment state on line 0.

That state remains active on lines 1 and 2.

After recognizing the closing delimiter on line 3, normal syntax resumes.

Conceptually:

```text
Line 0
START → IN_COMMENT

Line 1
IN_COMMENT → IN_COMMENT

Line 2
IN_COMMENT → IN_COMMENT

Line 3
IN_COMMENT → NORMAL

Line 4
NORMAL → NORMAL
```

This creates a dependency between lines.

If line 0 changes and no longer starts a comment, the highlighting of subsequent lines may also change.

Therefore, efficient highlighting requires more than simply reprocessing the line that was edited.

## Stable Syntax State

A useful approach is to associate each analyzed region with an input and output syntax state.

Conceptually:

```text
Line N

Input state
    │
    ▼
Analyze line
    │
    ▼
Highlight ranges
    │
    ▼
Output state
```

For example:

```text
Line 0:
Input  = NORMAL
Output = IN_COMMENT

Line 1:
Input  = IN_COMMENT
Output = IN_COMMENT

Line 2:
Input  = IN_COMMENT
Output = NORMAL
```

If an edit changes line 0, the editor can re-analyze subsequent lines until the resulting syntax state becomes stable again.

This gives a useful optimization without requiring a sophisticated incremental parser.

The details of efficient incremental updating will be discussed later in the chapter.

## Highlight Information Is Recomputable

The previous examples reveal an important property of syntax highlighting:

> **It is acceptable to discard and regenerate highlighting information.**

The source text is not lost because the Document remains authoritative.

For example:

```text
Document
   │
   ├── authoritative
   │
   ▼
Highlight cache
   │
   └── disposable
```

If the highlight cache becomes invalid:

```text
discard
   │
   ▼
re-analyze
   │
   ▼
rebuild
```

This makes syntax highlighting fundamentally different from information such as the user's actual text or file contents.

The syntax subsystem can trade memory and computation according to the editor's needs.

## A Clean Boundary With the Renderer

The final architecture can now be stated precisely.

The syntax subsystem owns:

```text
Language rules
Lexical analysis
Tokens
Syntax interpretation
Highlight classes
Highlight ranges
Syntax state
```

The Document owns:

```text
Source text
```

The Theme owns:

```text
Mapping from highlight classes
to visual styles
```

The Renderer owns:

```text
Visible frame construction
Screen-cell generation
Cursor presentation
Terminal output
```

Conceptually:

```text
             ┌──────────────┐
             │   Document   │
             └──────┬───────┘
                    │
                    ▼
             ┌──────────────┐
             │    Syntax    │
             │  subsystem  │
             └──────┬───────┘
                    │
             highlight classes
                    │
                    ▼
             ┌──────────────┐
             │    Theme     │
             └──────┬───────┘
                    │
               visual style
                    │
                    ▼
             ┌──────────────┐
             │   Renderer   │
             └──────┬───────┘
                    │
                    ▼
                Terminal
```

No component needs to take ownership of another component's responsibilities.

## The Rendering Path

Putting everything together, the editor's rendering path can be viewed as:

```text
                    Document
                       │
                       │ source text
                       ▼
                Syntax Subsystem
                       │
                       │ highlight classes
                       ▼
                     Theme
                       │
                       │ visual attributes
                       ▼
                    Renderer
                       │
                       │ screen cells
                       ▼
                 Screen Buffer
                       │
                       │ terminal output
                       ▼
                    Terminal
```

The Cursor and Viewport participate in determining which document region becomes visible:

```text
Document
   │
   ├── Syntax → highlight information
   │
   └── Cursor / Viewport → visible region
                              │
                              ▼
                           Renderer
```

This preserves the architecture established in earlier chapters while adding syntax highlighting as a new source of derived presentation information.

## Summary

Applying highlighting across a Document raises the practical problem of efficiency. Re-analyzing every line after every change is wasteful, so highlighting is often organized per line; but because syntax state can cross line boundaries, an edit can affect many later lines until the resulting state stabilizes again. This is acceptable precisely because highlighting is recomputable derived state: the Document remains authoritative and the syntax cache is disposable. The clear ownership boundaries—the syntax subsystem for understanding, the Theme for appearance, and the Renderer for the visible frame—keep each component focused without any component taking over another's responsibilities.

At this point, the editor can determine that a region is a keyword, comment, string, number, or other semantic category. But those categories still have no appearance, and the highlighting data must be connected to the text buffer that actually stores the source. The next section therefore examines **how syntax highlighting relates to the text buffer**: **Section 32.9, Syntax Highlighting and the Text Buffer**.
