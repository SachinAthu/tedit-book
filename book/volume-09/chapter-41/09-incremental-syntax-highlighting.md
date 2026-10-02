# 41.9 Incremental Syntax Highlighting

Syntax highlighting was introduced earlier as a lexical analysis problem. The editor can identify tokens such as keywords, identifiers, literals, and comments and then use that information when rendering the document.

Incremental parsing introduces another possibility.

Instead of deriving highlighting only from a stream of lexical tokens, the editor can derive highlighting from the **syntax tree** maintained by the parser.

The important architectural distinction is:

```text
Document
   │
   ▼
Syntax Analysis
   │
   ▼
Syntax Tree
   │
   ▼
Highlighting Information
   │
   ▼
Renderer
```

This allows syntax highlighting to take structural context into account.

## Why Lexical Highlighting Has Limits

Consider:

```c
id = 10;
```

A lexical highlighter can identify:

```text
id       identifier
=        operator
10       integer
;        punctuation
```

This is sufficient for many basic highlighting tasks.

However, some languages contain syntax whose interpretation depends on surrounding structure.

For example, the same sequence of characters may have different roles depending on whether it appears inside:

```text
a comment
a string
a type declaration
an expression
a preprocessor construct
```

A syntax tree provides additional context:

```text
Syntax Tree
     │
     ├── declaration
     ├── expression
     ├── string
     ├── comment
     └── other constructs
```

The highlighter can therefore make decisions using structural information rather than relying entirely on local character patterns.

## Highlighting From the Syntax Tree

The conceptual pipeline becomes:

```text
Document
    │
    ▼
Incremental Parser
    │
    ▼
Syntax Tree
    │
    ▼
Highlight Query / Analysis
    │
    ▼
Highlight Ranges
    │
    ▼
Renderer
```

The important point is that the syntax tree does not itself determine colors or terminal escape sequences.

It provides structural information.

A separate highlighting stage interprets that structure and produces information suitable for rendering.

This preserves the separation established earlier:

```text
Parser
  │
  └── structural analysis

Renderer
  │
  └── visual presentation
```

## Highlight Ranges

A useful representation of highlighting is a collection of source ranges associated with semantic categories.

Conceptually:

```text
Highlight
├── range
└── capture
```

For example:

```text
keyword     [0..3]
identifier  [4..9]
operator    [10..11]
number      [12..14]
```

The renderer can then map those categories to terminal presentation.

```text
Highlight Range
      │
      ▼
Renderer
      │
      ▼
ANSI styling
```

The parser therefore does not need to know which terminal escape sequence represents a particular visual style.

## Incremental Highlighting

The advantage of combining highlighting with incremental parsing is that an edit does not necessarily require the editor to reconsider the entire document.

Suppose:

```text
Document
├── Region A
├── Region B
├── Region C
└── Region D
```

An edit occurs in Region C.

The conceptual flow is:

```text
Edit
 │
 ▼
Updated Syntax Tree
 │
 ├── Region A → unchanged
 ├── Region B → unchanged
 ├── Region C → affected
 └── Region D → unchanged
```

The highlighting system can then update the ranges associated with the affected structure.

Conceptually:

```text
Changed syntax
      │
      ▼
Recalculate affected highlighting
      │
      ▼
Preserve unaffected highlighting
```

Again, this does **not** mean that only the changed character is analyzed. Structural changes can affect a larger region.

The important principle is that unchanged analysis does not need to be discarded without reason.

## A Small Edit Can Affect Highlighting Beyond One Character

Consider:

```c
"hello"
```

If the closing quotation mark is deleted:

```c
"hello
```

the effect is not limited to that one character.

The parser may now interpret subsequent source differently because the string has not been closed.

For example:

```c
"hello
int value = 10;
```

The missing delimiter can affect how the following text is interpreted.

This illustrates the same principle established for incremental parsing:

```text
Small text edit
       │
       ▼
Potentially larger syntactic effect
       │
       ▼
Affected highlighting region
```

Therefore, incremental highlighting should follow the syntax structure rather than assuming that highlighting changes are always character-local.

## Syntax Tree as Shared Information

The syntax tree can support several features simultaneously:

```text
                    Syntax Tree
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
     Highlighting      Folding      Navigation
```

This is one of the main architectural benefits of maintaining structural analysis.

Without a syntax tree, each feature may need to perform its own independent interpretation of the source.

With a syntax tree, structural information can be shared.

However, the features should still remain separate.

The syntax tree is shared information, not a reason to combine all feature logic into one subsystem.

## Highlighting Is Derived State

As with the syntax tree itself, highlighting information is derived from the Document.

The relationship is:

```text
Document
   │
   ▼
Syntax Tree
   │
   ▼
Highlight Information
   │
   ▼
Renderer
```

None of these derived layers becomes authoritative.

If the Document changes:

```text
Document
   │
   │ edit
   ▼
new Document state
   │
   ▼
updated syntax tree
   │
   ▼
updated highlighting
```

This keeps the ownership model clear.

## Highlighting and the Renderer

The Renderer should not become responsible for understanding the programming language.

Its responsibility remains presentation.

Conceptually:

```text
Syntax Analysis
      │
      ▼
Highlight Information
      │
      ▼
Renderer
      │
      ▼
ANSI Escape Sequences
      │
      ▼
Terminal
```

The Renderer receives information about how source regions should be displayed.

It does not need to understand what a C function, expression, or declaration means.

This follows the architectural decision to keep language analysis separate from terminal rendering.

## Syntax Highlighting Does Not Modify Source

The distinction between derived information and document state is especially important here.

Highlighting may visually distinguish:

```text
keyword
identifier
string
comment
number
```

but those categories do not become properties stored inside the Document itself.

The Document remains:

```text
int value = 10;
```

The highlighting subsystem derives:

```text
int       → keyword/type
value     → identifier
10        → number
```

The Renderer turns those classifications into visual output.

This prevents presentation metadata from contaminating the core text model.

## Interaction With the Screen Buffer

The highlighting stage also fits naturally into the rendering pipeline established earlier.

Conceptually:

```text
Document
   │
   ▼
Syntax Tree
   │
   ▼
Highlight Information
   │
   ▼
Screen Buffer
   │
   ▼
Renderer
   │
   ▼
Terminal
```

The Screen Buffer represents what should appear on the terminal.

The syntax-analysis subsystem determines how source regions should be represented.

The Renderer ultimately converts that information into terminal output.

The responsibilities therefore remain distinct:

```text
Document       → source
Syntax Tree    → structure
Highlighting   → presentation metadata
Screen Buffer  → rendered frame
Renderer       → terminal output
```

## Incremental Highlighting and Scrolling

Scrolling does not require re-parsing the document.

The syntax tree represents the document independently of the current viewport:

```text
Document
   │
   ▼
Syntax Tree
```

The Viewport determines which portion is visible:

```text
Syntax / Document
       │
       ▼
Viewport
       │
       ▼
Visible source region
```

Highlighting information can therefore remain associated with document positions while the viewport moves.

```text
Syntax Tree
     │
     ▼
Highlight ranges
     │
     ▼
Viewport selects visible ranges
     │
     ▼
Renderer
```

This preserves the separation between document coordinates and terminal coordinates established earlier.

## Incremental Highlighting vs Full Redraw

Our renderer uses a full-redraw strategy.

That decision does not conflict with incremental syntax highlighting.

These concepts operate at different levels.

**Incremental highlighting** determines how efficiently the editor maintains semantic presentation information.

**Full redraw** determines how the Renderer produces the terminal frame.

The architecture can therefore be:

```text
             Document
                 │
                 ▼
          Incremental Parser
                 │
                 ▼
            Syntax Tree
                 │
                 ▼
        Highlight Information
                 │
                 ▼
           Screen Buffer
                 │
                 ▼
             Renderer
                 │
                 ▼
        Full Terminal Redraw
```

The Renderer may still redraw the visible screen completely even though the syntax analysis was updated incrementally.

There is no contradiction.

## Highlighting and Incomplete Code

The previous section established that the parser must tolerate incomplete source.

That behavior is equally important for highlighting.

Consider:

```c
printf("hello
```

The user has not finished the string.

The editor should not stop rendering simply because the syntax is incomplete.

Instead, the parser provides the best structural information available, and the highlighting layer derives presentation from that information.

```text
Incomplete source
       │
       ▼
Error-tolerant parser
       │
       ▼
Partial syntax structure
       │
       ▼
Highlighting
       │
       ▼
Renderer
```

As the user types the closing quotation mark, the syntax tree and highlighting can be updated again.

## Why This Is Different From Chapter 32

Chapter 32 established syntax highlighting using lexical analysis.

The important distinction here is not that the earlier approach was wrong.

It is that the two approaches operate at different levels.

Earlier:

```text
Document
   │
   ▼
Lexical Analysis
   │
   ▼
Tokens
   │
   ▼
Highlighting
```

With incremental syntax analysis:

```text
Document
   │
   ▼
Incremental Parser
   │
   ▼
Syntax Tree
   │
   ▼
Highlighting
```

The second approach can use structural context that is not available from a simple token stream.

This chapter therefore extends the earlier design rather than replacing the conceptual foundation introduced in Chapter 32.

## The Performance Goal

The objective is not:

> Make syntax highlighting infinitely fast.

The practical goal is:

> Avoid unnecessarily recomputing highlighting information that remains valid after an edit.

The conceptual comparison is:

```text
Full recomputation:

Edit
 │
 ▼
Analyze entire document
 │
 ▼
Rebuild all highlighting
```

versus:

```text
Incremental:

Edit
 │
 ▼
Update syntax structure
 │
 ▼
Identify affected analysis
 │
 ▼
Update affected highlighting
```

The second approach can reduce unnecessary work for ordinary local edits.

## The Important Boundary

We should preserve a clear boundary between the parser and highlighting subsystem:

```text
Tree-sitter
    │
    └── maintains syntax structure

Highlighting
    │
    └── interprets syntax structure

Renderer
    │
    └── presents highlighting
```

This avoids turning the parser into a presentation system or the Renderer into a programming-language analyzer.

## The Resulting Architecture

The complete flow is now:

```text
                         Document
                            │
                            │ edit
                            ▼
                   Incremental Parser
                            │
                            ▼
                       Syntax Tree
                            │
                            ▼
                   Highlight Analysis
                            │
                            ▼
                     Highlight Ranges
                            │
                            ▼
                       Screen Buffer
                            │
                            ▼
                         Renderer
                            │
                            ▼
                     ANSI / Terminal
```

Each stage has a distinct responsibility.

The Document owns the source.

The parser derives structure.

The highlighting subsystem derives presentation metadata.

The Screen Buffer and Renderer turn that information into the visible terminal frame.
