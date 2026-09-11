# 32.7 Lightweight Syntax Parsing

Lexical analysis can identify the pieces of a source file, but some highlighting decisions depend on relationships between those pieces.

A lexer might tell us:

```text
int        → KEYWORD
calculate  → IDENTIFIER
(          → PUNCTUATION
int        → KEYWORD
value      → IDENTIFIER
)          → PUNCTUATION
```

That is enough to recognize the basic lexical structure.

It is not necessarily enough to determine that:

```text
calculate
```

is being declared as a function.

A lightweight syntax parser provides a limited amount of structural understanding without requiring the editor to implement a complete compiler.

## What "Lightweight" Means

A lightweight parser does not attempt to understand every aspect of a programming language.

Its purpose is narrower:

> **Recognize enough structural relationships to improve syntax highlighting.**

For example, it might recognize that:

```text
int calculate(int value)
```

contains:

```text
TYPE       → int
FUNCTION   → calculate
PARAMETER  → value
```

It does not need to determine whether the function is correctly implemented, whether its types are compatible, or whether another module defines a referenced symbol.

The parser is therefore concerned with **highlighting-relevant structure**, not complete program semantics.

## Parsing Tokens Rather Than Characters

The parser operates on tokens produced by the lexer.

The pipeline becomes:

```text
Document
    │
    ▼
  Lexer
    │
    ▼
 Tokens
    │
    ▼
Lightweight Parser
    │
    ▼
Highlight Classes
```

This separation is useful because the parser does not need to rediscover lexical boundaries.

The lexer has already determined that:

```text
calculate
```

is an identifier and that:

```text
(
```

is punctuation.

The parser can concentrate on relationships between those tokens.

For example:

```text
IDENTIFIER
    │
    │ followed by "("
    ▼
Possible function-related construct
```

The parser can then assign a more specific highlight class when the surrounding structure supports that interpretation.

## Recognizing Simple Structures

Consider:

```text
int calculate(int value) {
    return value + 10;
}
```

A lightweight parser could recognize the broad structure:

```text
Declaration
├── Type
│   └── int
├── Function name
│   └── calculate
├── Parameter list
│   └── value
└── Body
```

It does not need to build a complete representation of every expression inside the function.

For highlighting purposes, recognizing the important regions may be sufficient:

```text
int        → TYPE
calculate  → FUNCTION
int        → TYPE
value      → PARAMETER
return     → KEYWORD
value      → VARIABLE
10         → NUMBER
```

The parser has improved the semantic classification without becoming responsible for the entire language.

## A Small State Machine Can Sometimes Be Enough

Not every contextual rule requires a traditional parser.

Some languages or highlighting rules can be handled with a small state machine.

For example, after seeing a type-like token followed by an identifier and an opening parenthesis:

```text
TYPE IDENTIFIER (
```

the analyzer may tentatively interpret the identifier as a function name.

Conceptually:

```text
START
  │
  │ TYPE
  ▼
AFTER_TYPE
  │
  │ IDENTIFIER
  ▼
POSSIBLE_DECLARATION
  │
  │ "("
  ▼
FUNCTION_LIKE_DECLARATION
```

This is substantially simpler than parsing the entire language grammar.

The appropriate technique depends on how much context the language requires.

## Grammar as a Source of Structure

When rules become more complicated, a grammar provides a more formal way to describe relationships between tokens.

For example, a simplified grammar might describe:

```text
function_declaration
    → type identifier "(" parameter_list ")"
```

This tells the parser that a sequence of tokens matching this pattern represents a particular structural construct.

The grammar does not need to model every language feature to be useful for highlighting.

A deliberately reduced grammar can focus only on structures relevant to visual classification.

This is an important engineering trade-off:

```text
Complete language grammar
        │
        ├── more accurate
        ├── more implementation effort
        └── more maintenance
        │
        ▼
Reduced highlighting grammar
        │
        ├── simpler
        ├── faster to implement
        └── sufficient for many visual distinctions
```

The editor should choose the smallest useful model.

## Error Recovery Matters

Unlike a compiler, an editor must operate while the source code is incomplete.

This is one of the most important differences between editor parsing and traditional compilation.

A user may temporarily have:

```text
int calculate(
```

or:

```text
if (count > 10 {
```

or:

```text
"unfinished string
```

These are normal editing states.

A parser that expects perfect source code would frequently fail while the user is typing.

A syntax highlighter therefore needs some form of graceful recovery.

Conceptually:

```text
Valid syntax
     │
     ▼
Normal analysis
     │
     ▼
Highlighting

Incomplete syntax
     │
     ▼
Best-effort analysis
     │
     ▼
Useful highlighting
```

The goal is not to reject malformed input.

The goal is to continue producing the best highlighting information available.

## Best-Effort Parsing

Suppose the user is typing:

```text
int calculate(int val
```

The closing parenthesis has not been entered yet.

A useful highlighter can still recognize:

```text
int       → TYPE
calculate → FUNCTION
int       → TYPE
val       → PARAMETER
```

It does not need to declare the entire construct invalid.

This suggests an important editor-specific principle:

> **Syntax highlighting should degrade gracefully when syntax is incomplete or temporarily invalid.**

This principle also applies to partially typed strings, comments, declarations, and expressions.

The editor should remain responsive and visually useful throughout the editing process.

## Parsing Does Not Own the Document

The lightweight parser reads the Document through the syntax-analysis pipeline.

It does not modify the source text.

The relationship remains:

```text
             ┌──────────────┐
             │   Document   │
             └──────┬───────┘
                    │
                    │ read
                    ▼
              ┌───────────┐
              │   Lexer   │
              └─────┬─────┘
                    │
                    ▼
               ┌────────┐
               │ Tokens │
               └────┬───┘
                    │
                    ▼
              ┌───────────┐
              │  Parser   │
              └─────┬─────┘
                    │
                    ▼
           Highlight Classes
```

The parser should not:

- modify Document lines,
- move the Cursor,
- change the Viewport,
- produce terminal escape sequences,
- draw directly to the terminal,
- perform file I/O,
- or decide theme styles.

Its output is syntax information.

This preserves the architecture established throughout the editor.

## Parser Output Does Not Have to Be a Full Syntax Tree

A common assumption is that parsing necessarily means constructing a complete abstract syntax tree.

That is not required here.

A lightweight highlighter could produce only the information it needs:

```text
FUNCTION
 └── range

PARAMETER
 └── range

TYPE
 └── range
```

Instead of:

```text
Complete syntax tree
├── Translation unit
├── Declaration
│   ├── Type
│   ├── Declarator
│   ├── Parameter list
│   └── ...
└── ...
```

The smaller representation can be easier to maintain and may require less memory.

A complete syntax tree becomes valuable when many other editor features need structural information.

That is a broader architectural decision and belongs beyond the initial syntax-highlighting subsystem.

## Choosing the Right Analysis Level

There are several possible implementation levels:

```text
Level 1 — Lexical highlighting

Document
   ↓
Lexer
   ↓
Highlight Classes
```

```text
Level 2 — Lightweight contextual highlighting

Document
   ↓
Lexer
   ↓
Tokens
   ↓
Contextual Rules
   ↓
Highlight Classes
```

```text
Level 3 — Parser-based highlighting

Document
   ↓
Lexer
   ↓
Parser
   ↓
Syntax Structure
   ↓
Highlight Classes
```

```text
Level 4 — Full language infrastructure

Document
   ↓
Incremental Parser
   ↓
Syntax Tree
   ↓
Semantic Information
   ↓
Highlight Classes
```

Each level provides more structural information but also introduces additional complexity.

The editor should not jump to Level 4 simply because it produces the most sophisticated result.

A professional architecture should make it possible to evolve from one level to another without requiring unrelated components to be redesigned.

## The Importance of Isolation

The syntax parser should be isolated behind a clear interface.

Conceptually:

```text
             Syntax Subsystem
┌────────────────────────────────────┐
│                                    │
│  Language Definition               │
│          │                         │
│          ▼                         │
│        Lexer                       │
│          │                         │
│          ▼                         │
│       Tokens                       │
│          │                         │
│          ▼                         │
│  Lightweight Analysis              │
│          │                         │
│          ▼                         │
│  Highlight Ranges                  │
│                                    │
└──────────────────┬─────────────────┘
                   │
                   ▼
                Renderer
```

The rest of the editor does not need to know how the syntax subsystem arrived at its result.

This allows the implementation to change later.

For example, a simple contextual analyzer could eventually be replaced with a more capable parser while preserving the same conceptual output:

```text
Highlight Ranges
```

The Renderer does not need to care whether those ranges came from a state machine, a grammar-based parser, or a more sophisticated syntax engine.

## Avoiding Overengineering

There is a natural temptation to make syntax highlighting extremely sophisticated.

Modern editors can provide:

- detailed syntax trees,
- semantic classification,
- incremental parsing,
- language-server integration,
- project-wide symbol information,
- semantic tokens,
- and other advanced features.

These capabilities are valuable, but they solve problems beyond basic syntax highlighting.

For the editor in this book, the syntax subsystem should first establish a clean foundation:

```text
Document
   ↓
Language-aware analysis
   ↓
Highlight information
   ↓
Renderer
```

The architecture should leave room for future sophistication without requiring it in the initial implementation.

This is a recurring engineering principle:

> **Design stable boundaries before optimizing for advanced capabilities.**

## Syntax Parsing Is Still Derived Information

Even after introducing contextual parsing, the ownership model does not change.

The Document remains authoritative:

```text
Document
   │
   ├── source text
   │
   └── authoritative state
```

The syntax subsystem produces:

```text
Tokens
Highlight Ranges
```

These are derived from the current Document.

If the Document changes, they may need to be recalculated.

This distinction will become particularly important when considering edits.

For example:

```text
User edits Document
        │
        ▼
Affected syntax becomes stale
        │
        ▼
Re-analyze
        │
        ▼
Updated highlighting
```

How much of the Document needs to be re-analyzed is an important performance question, but the basic ownership model remains unchanged.

## The Boundary Before Rendering

At this point, the syntax subsystem can produce semantic information such as:

```text
KEYWORD
FUNCTION
VARIABLE
STRING
COMMENT
NUMBER
OPERATOR
```

The renderer still does not need to understand what those categories mean.

It only needs to know that a particular document range has a particular highlight class.

The final transformation is therefore:

```text
Document
   │
   ▼
Syntax Analysis
   │
   ▼
Highlight Ranges
   │
   ▼
Theme
   │
   ▼
Visual Attributes
   │
   ▼
Renderer
```

This is the boundary that keeps language knowledge out of the rendering system.

The syntax subsystem determines **what the text represents**.

The theme determines **how that category should look**.

The renderer determines **how to place that styled text into the current screen frame**.

## A Useful Mental Model

The entire syntax pipeline can now be understood as a sequence of increasingly meaningful representations:

```text
Characters
    │
    ▼
Tokens
    │
    ▼
Structural relationships
    │
    ▼
Highlight Classes
    │
    ▼
Visual Styles
```

Each stage adds information without replacing the previous authoritative layer.

This gives the editor a clean conceptual model:

> **Source text is data. Tokens describe lexical structure. Lightweight parsing adds context. Highlight classes describe semantic presentation categories. Themes translate those categories into visual styles.**

The next question is how these highlighting decisions are actually applied across the Document.

## Summary

Lightweight syntax parsing is a deliberately limited form of structural analysis that improves highlighting by reasoning about the relationships between tokens—for example, telling when an identifier names a function rather than a variable—without attempting to understand the entire program. It stays well short of a compiler or language server, yet it lets the subsystem produce highlight ranges that keep language knowledge out of the rendering system. Like all syntax information, the results are derived from the current Document and can be recalculated when the source changes, and the Document remains authoritative.

A syntax subsystem may understand the entire source file, but the renderer only displays a viewport-sized portion of it at any given moment.

The next section therefore examines **how syntax highlighting is applied to the Document and connected to the rendering process**: **Section 32.8, Highlighting the Document**.
