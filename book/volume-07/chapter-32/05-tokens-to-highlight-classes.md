# 32.5 From Tokens to Highlight Classes

Tokens tell us what kind of lexical elements exist in the source text. However, lexical token types are not necessarily the right abstraction for deciding how text should be highlighted.

A syntax-highlighting system therefore introduces another layer: the **highlight class**.

A highlight class is a semantic category that describes how a piece of source text should be treated by the presentation layer.

The distinction is subtle but important:

```text
Token Type
    │
    │ semantic interpretation
    ▼
Highlight Class
    │
    │ visual mapping
    ▼
Theme Style
```

This separation prevents the lexer from becoming coupled to a particular visual design.

## Why Token Types Are Not Enough

Consider this source:

```text
int calculate(int value) {
    return value + 10;
}
```

A lexer can identify:

```text
int          → KEYWORD
calculate    → IDENTIFIER
(            → PUNCTUATION
int          → KEYWORD
value        → IDENTIFIER
)            → PUNCTUATION
{            → PUNCTUATION
return       → KEYWORD
value        → IDENTIFIER
+            → OPERATOR
10           → NUMBER
;            → PUNCTUATION
}            → PUNCTUATION
```

This is useful, but it does not completely describe the semantic role of each identifier.

For example:

```text
calculate
```

could be an ordinary variable, a function name, a type name, or another language-specific construct.

The lexer alone may not be able to determine that distinction.

A later syntax-analysis stage can refine the information:

```text
IDENTIFIER
     │
     ├── VARIABLE
     ├── FUNCTION
     ├── TYPE
     └── OTHER
```

The result is a more useful representation for highlighting.

## Highlight Classes Are Semantic, Not Visual

A highlight class should describe a semantic category rather than a specific terminal appearance.

For example:

```text
keyword
string
number
comment
variable
function
type
operator
punctuation
```

These names describe what the source element represents.

They do **not** say:

```text
blue
green
bold
underline
bright
```

That distinction belongs to the theme.

A theme might map:

```text
keyword  → one visual style
string   → another visual style
comment  → another visual style
```

Another theme can map the same classes completely differently.

Therefore:

```text
Syntax subsystem
    │
    └── "This is a keyword."

Theme
    │
    └── "Keywords should look like this."

Renderer
    │
    └── "Apply that style while producing the frame."
```

Each layer has one clear responsibility.

## A Mapping Layer

The relationship between token types and highlight classes can be simple.

For many languages:

```text
KEYWORD       → KEYWORD
NUMBER        → NUMBER
STRING        → STRING
COMMENT       → COMMENT
OPERATOR      → OPERATOR
PUNCTUATION   → PUNCTUATION
```

But other cases require additional interpretation:

```text
IDENTIFIER
    │
    ├── VARIABLE
    ├── FUNCTION
    └── TYPE
```

This gives the syntax subsystem room to become more expressive without forcing the lexer itself to understand every semantic distinction.

The overall pipeline becomes:

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
 Syntax Analysis
   │
   ▼
 Highlight Classes
   │
   ▼
 Theme
   │
   ▼
 Renderer
```

The lexer identifies structure.

Syntax analysis interprets that structure.

Highlight classes represent the resulting semantic categories.

The theme determines their appearance.

## Why This Separation Matters

Suppose the editor initially supports only a small set of visual categories:

```text
keyword
string
comment
number
normal
```

Later, it may become useful to distinguish:

```text
function
type
constant
variable
parameter
attribute
```

If lexical token types are directly tied to terminal colors, introducing these categories would require changes throughout the system.

With highlight classes, the layers remain independent.

The lexer can continue producing:

```text
IDENTIFIER
```

while the syntax analysis layer determines:

```text
FUNCTION
```

and the theme decides how `FUNCTION` should appear.

This makes the architecture easier to extend.

## Highlighting Is Not Limited to Colors

Although syntax highlighting is commonly associated with colors, a highlight class can represent any visual attribute supported by the editor's presentation system.

For example:

```text
COMMENT
    → subdued foreground

KEYWORD
    → emphasized foreground

ERROR
    → warning-style appearance

FUNCTION
    → distinct foreground

NORMAL
    → default appearance
```

The exact visual attributes are intentionally outside the syntax-analysis layer.

This is important because the same semantic information could eventually be presented in different ways.

For example, a terminal theme might use foreground colors, while another presentation environment could use different visual attributes.

The syntax subsystem should not need to know which mechanism is being used.

## The Default Highlight Class

Not every character needs a special semantic category.

Most source text can simply use a default class:

```text
NORMAL
```

For example:

```text
int count = 42;
```

might produce:

```text
int       → KEYWORD
count     → VARIABLE
=         → OPERATOR
42        → NUMBER
;         → PUNCTUATION
```

Any text not covered by a more specific classification remains:

```text
NORMAL
```

This gives the highlighting system a useful default state.

Conceptually:

```text
Document region
      │
      ├── classified → specific Highlight Class
      │
      └── unclassified → NORMAL
```

The default class also simplifies rendering because the renderer does not need a special case for every character that lacks syntax information.

## Highlight Ranges

A practical syntax-highlighting system usually needs to associate a highlight class with a region of the Document.

Conceptually:

```text
Highlight Range
 ├── Start
 ├── End
 └── Class
```

For:

```text
int count = 42;
```

the derived information might look like:

```text
(0, 0) → (0, 3)   KEYWORD
(0, 4) → (0, 9)   VARIABLE
(0,10) → (0,11)   OPERATOR
(0,12) → (0,14)   NUMBER
(0,14) → (0,15)   PUNCTUATION
```

The ranges remain document-relative.

They do not change simply because the user scrolls horizontally or vertically.

The renderer can later translate those logical document positions into screen positions while constructing the current frame.

This maintains the separation between **document coordinates** and **screen coordinates**.

## One Character Can Be Classified Only Once

For a simple highlighter, each document position normally receives one effective highlight class.

For example:

```text
"hello"
```

might be classified as:

```text
STRING
```

rather than simultaneously being:

```text
STRING
IDENTIFIER
KEYWORD
```

The syntax subsystem therefore needs a rule for resolving overlapping classifications when multiple analysis stages identify the same region.

A common conceptual approach is to establish precedence:

```text
comment
    ↓
string
    ↓
keyword
    ↓
number
    ↓
identifier
    ↓
operator
    ↓
punctuation
    ↓
normal
```

The exact precedence depends on the language.

The important idea is that highlighting should produce a deterministic result.

For example, the text inside a string literal should normally remain part of the string even if it happens to contain text that resembles a keyword:

```text
"return value"
```

The word `return` is not a keyword here. It is part of a string.

This is one reason lexical state and contextual syntax analysis matter.

## Highlight Classes Should Remain Stable

A highlight class should have a stable semantic meaning.

For example:

```text
KEYWORD
```

should mean "language keyword," not:

```text
whatever the current theme displays in blue
```

Similarly:

```text
COMMENT
```

should mean "comment text," not:

```text
dim gray text
```

This stability makes themes interchangeable.

A theme can change from:

```text
KEYWORD → bright style
```

to:

```text
KEYWORD → subdued style
```

without requiring the syntax analyzer to change.

That is one of the main architectural benefits of the additional abstraction.

## Syntax Information Is Still Derived State

Highlight classes do not become part of the Document itself.

The editor still has one authoritative representation of the source:

```text
Document
```

The syntax subsystem derives:

```text
Tokens
Highlight Classes
Highlight Ranges
```

from it.

Therefore:

```text
                authoritative
                    │
                    ▼
               ┌──────────┐
               │ Document │
               └────┬─────┘
                    │
             derived information
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       Tokens          Highlight Ranges
                              │
                              ▼
                            Theme
                              │
                              ▼
                           Renderer
```

If the Document changes, the derived syntax information may become stale.

It can be discarded and regenerated.

This is preferable to embedding highlighting information directly into the text buffer because it keeps document storage independent from language-specific analysis.

## A Useful Architectural Boundary

The syntax subsystem should answer questions such as:

- Where are the language constructs?
- Which regions are comments?
- Which regions are strings?
- Which identifiers represent functions?
- Which regions should receive the `KEYWORD` highlight class?

It should not answer questions such as:

- What ANSI escape sequence represents this class?
- Which screen row contains this token?
- Where should the cursor be drawn?
- How should the terminal be cleared?
- Which theme file should the user configure?

Those responsibilities belong elsewhere.

The separation can therefore be summarized as:

```text
Syntax
  "What does this text represent?"

Theme
  "How should that category look?"

Renderer
  "How do I display it in the current frame?"
```

Keeping these questions separate is what allows syntax highlighting to grow without turning the editor into a tightly coupled collection of language-specific rendering code.

## From Highlight Classes to Visual Styles

At this point, the syntax subsystem has transformed raw document text into semantic information:

```text
Source text
    │
    ▼
Tokens
    │
    ▼
Highlight Classes
```

The remaining question is visual rather than syntactic:

> How should each highlight class appear on the terminal?

That decision belongs to the **theme**.

The theme creates the final mapping:

```text
Highlight Class
      │
      ▼
Visual Style
```

For example:

```text
KEYWORD      → style A
STRING       → style B
COMMENT      → style C
NUMBER       → style D
NORMAL       → default style
```

The syntax subsystem does not need to know what those styles actually contain.

## Summary

Highlight classes are semantic categories that sit between lexical analysis and presentation. They are distinct from token types: a token describes what the text is syntactically, while a highlight class describes which semantic presentation category applies, letting syntax analysis and visual design evolve independently. Classification uses document-relative ranges, a NORMAL default class for unclassified text, and deterministic precedence to resolve overlapping classifications. Because highlight classes keep stable meanings, Themes can be swapped without forcing language analysis to change, and the resulting syntax information remains derived state that can be discarded and regenerated from the Document.

This completes the conceptual separation between language analysis and presentation, and leads naturally to the next part of the chapter: determining when lexical analysis alone is insufficient and why some syntax requires contextual understanding. That is the subject of **Section 32.6, Why Lexing Alone Is Not Enough**.
