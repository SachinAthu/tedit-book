# 32.4 Tokens

The lexer identifies meaningful pieces of source text, but those pieces need a representation that the rest of the syntax-highlighting system can work with. A **token** provides that representation.

A token describes a region of the Document and gives that region a lexical or syntactic meaning. It does not replace the text in the Document, and it does not contain visual formatting. Instead, it acts as a reference to derived information about existing document text.

For example, given:

```text
int count = 42;
```

the lexer might produce:

```text
KEYWORD    "int"
IDENTIFIER "count"
OPERATOR   "="
NUMBER     "42"
PUNCTUATION ";"
```

The important point is that the token list is **derived from the Document**. The actual characters still belong to the Document.

## A Token Is a Typed Region

A useful conceptual model is:

```text
Token
 ├── Type
 ├── Start position
 └── Length
```

The **type** identifies what kind of lexical element the token represents.

The **start position** identifies where the token begins in the Document.

The **length** identifies how much document text belongs to the token.

Conceptually:

```text
Document:

int count = 42;
^^^
│
└── token starts here

Token:

Type   = KEYWORD
Start  = (row 0, column 0)
Length = 3
```

The token does not need to copy `"int"` into a separate string. The text can be obtained from the referenced region of the Document when needed.

This distinction is important because syntax highlighting can produce a large amount of derived information. Duplicating the text inside every token would unnecessarily increase memory usage and complicate ownership.

## Token Positions

The position stored by a token must use the same logical coordinate system as the Document.

In this editor, the Document is organized as lines of Unicode code points. Therefore, syntax information should describe locations in terms of the Document's logical positions rather than terminal screen coordinates or encoded file bytes.

For a simple token:

```text
int count = 42;
```

the tokens could be represented conceptually as:

| Token   |     Start | Length |
| ------- | --------: | -----: |
| `int`   |  `(0, 0)` |      3 |
| `count` |  `(0, 4)` |      5 |
| `=`     | `(0, 10)` |      1 |
| `42`    | `(0, 12)` |      2 |
| `;`     | `(0, 14)` |      1 |

The exact internal structure can vary, but the important architectural rule is that token positions must remain meaningful with respect to the Document.

This also prevents an important category of coupling.

A token should **not** be defined using:

```text
terminal row
terminal column
screen cell
viewport position
```

Those are presentation concepts. Syntax belongs to the logical document, not to the current viewport.

## Start and Length Versus Start and End

There are several ways to represent a token's range.

One possibility is:

```text
start + length
```

Another is:

```text
start + end
```

Both describe a range of document text.

For example:

```text
start = column 4
length = 5
```

describes the same region as:

```text
start = column 4
end   = column 9
```

The choice is primarily an internal representation decision.

A start-and-length representation can be convenient when the lexer naturally consumes a known number of characters. A start-and-end representation can make range comparisons and transformations easier in some implementations.

For multi-line constructs, however, a single column value is insufficient. A token representing a block comment could begin on one line and end several lines later:

```text
/*
   explanation
   continues here
*/
```

Conceptually:

```text
Start: (10, 4)
End:   (12, 2)
```

The token representation therefore needs to be capable of describing ranges that cross line boundaries if the language definition permits multi-line lexical constructs.

The important principle is not which representation is chosen, but that a token must precisely identify its region in the logical Document.

## Tokens Should Not Own Document Text

A tempting design is to make every token contain its own copy of the text:

```text
Token
 ├── type
 ├── start
 ├── length
 └── text
```

For example:

```text
KEYWORD    → "int"
IDENTIFIER → "count"
NUMBER     → "42"
```

This appears convenient, but it creates unnecessary duplication.

The Document already contains:

```text
int count = 42;
```

Copying every token's text means the same information exists in multiple places.

A better model is:

```text
             ┌──────────────┐
             │   Document   │
             │              │
             │ int count... │
             └──────┬───────┘
                    │
                    │ references
                    ▼
             ┌──────────────┐
             │    Tokens    │
             │              │
             │ type         │
             │ start        │
             │ length       │
             └──────────────┘
```

The Document remains the authoritative source of text.

Tokens are only an interpretation of that text.

This follows the same broader architectural principle used throughout the editor: **derived state should not become a second authoritative copy of the underlying data**.

## Token Types and Highlight Classes Are Different

It is also important not to confuse **token types** with **highlight classes**.

A token type describes the role of a piece of source text at the lexical level:

```text
KEYWORD
IDENTIFIER
NUMBER
STRING
COMMENT
OPERATOR
```

A highlight class describes how the syntax-highlighting system wants to treat that semantic category visually:

```text
keyword
variable
number
string
comment
operator
```

These concepts are related, but they are not necessarily identical.

For example:

```text
Token type:
IDENTIFIER

Highlight class:
VARIABLE
```

or:

```text
Token type:
IDENTIFIER

Highlight class:
FUNCTION
```

A more context-aware syntax analyzer may determine that an identifier represents a function name rather than an ordinary variable.

The distinction can therefore be viewed as:

```text
Document text
      │
      ▼
    Token
      │
      ▼
Highlight Class
      │
      ▼
    Theme
      │
      ▼
Visual Style
```

The token describes **what the text is syntactically**.

The highlight class describes **which semantic visual category should be applied**.

The theme then decides what that category looks like.

This separation allows the visual design to change without changing the lexical analysis.

## Token Lifetime

Tokens are derived from the current contents of the Document.

That means they have a different lifetime from the Document itself.

Consider:

```text
int count = 42;
```

The lexer produces tokens for that text.

Now suppose the user changes the document:

```text
int total = 42;
```

The token describing `count` is no longer valid because the underlying document region has changed.

Therefore, syntax information must be treated as **recomputable derived state**.

Conceptually:

```text
Document changes
       │
       ▼
Old syntax information
       │
       └── becomes stale

       ▼

Re-analyze affected text

       ▼

New tokens
```

This is especially important when designing token storage.

A token should not be treated as permanent document metadata. It is valid only for the document state from which it was produced.

Later sections will examine how the editor can update this derived information efficiently after edits.

## Token Storage

A syntax subsystem can store tokens explicitly:

```text
Token 0
Token 1
Token 2
Token 3
...
```

or process them as a stream:

```text
Lexer
  │
  ├── Token
  ├── Token
  ├── Token
  └── ...
```

Storing tokens is useful when later stages need to inspect them repeatedly, such as when syntax analysis requires contextual information.

Streaming can reduce intermediate storage when the processing pipeline can consume each token immediately.

The appropriate choice depends on the complexity of the syntax subsystem.

For the architecture developed in this book, the important requirement is that token storage remains a **derived representation** rather than becoming another text buffer.

If all tokens are stored, the additional memory requirement is generally proportional to the number of tokens:

```text
O(t)
```

where `t` is the number of stored tokens.

The underlying Document remains the source of the actual text.

## A Complete Example

Consider:

```text
// Calculate the total
int total = count + 10;
```

A lexer might identify regions such as:

| Text                     | Token type  |     Start | Length |
| ------------------------ | ----------- | --------: | -----: |
| `// Calculate the total` | COMMENT     |  `(0, 0)` |     23 |
| `int`                    | KEYWORD     |  `(1, 0)` |      3 |
| `total`                  | IDENTIFIER  |  `(1, 4)` |      5 |
| `=`                      | OPERATOR    | `(1, 10)` |      1 |
| `count`                  | IDENTIFIER  | `(1, 12)` |      5 |
| `+`                      | OPERATOR    | `(1, 18)` |      1 |
| `10`                     | NUMBER      | `(1, 20)` |      2 |
| `;`                      | PUNCTUATION | `(1, 22)` |      1 |

The tokens provide a structured description of the source text:

```text
Document
│
├── COMMENT
│
└── Statement
    ├── KEYWORD
    ├── IDENTIFIER
    ├── OPERATOR
    ├── IDENTIFIER
    ├── OPERATOR
    ├── NUMBER
    └── PUNCTUATION
```

At this stage, however, the system still has not decided exactly how these regions should look.

The lexer has identified the pieces.

The next stage must determine their highlighting semantics.

## Common Token Design Mistakes

Several mistakes are particularly easy to make.

### 1. Copying all text into tokens

This duplicates information already owned by the Document and increases memory usage.

### 2. Storing terminal coordinates

Terminal coordinates describe the rendered frame, not the source document. Scrolling would make such coordinates immediately unsuitable for syntax data.

### 3. Storing ANSI escape sequences

A token should not contain terminal formatting such as color escape sequences. That would couple lexical analysis directly to terminal rendering.

### 4. Treating tokens as permanent

Editing the Document can invalidate token ranges. Syntax information must therefore be refreshed or invalidated when its source text changes.

### 5. Mixing lexical meaning with visual appearance

A token should not need to know whether a keyword is displayed in a particular color or with a particular terminal attribute.

These mistakes all violate the same architectural principle:

> **Syntax analysis describes the Document; rendering decides how that description is displayed.**

## The Role of Tokens in the Pipeline

Tokens provide the bridge between raw source text and higher-level syntax meaning.

The complete relationship is now:

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

The lexer answers:

> What lexical regions exist in this text?

Tokens record the answer in a structured form.

## Summary

A token is a typed reference to a region of the Document, described by a type, a start position, and a length, rather than a copy of the text. Tokens use the Document's logical coordinate system, not terminal or viewport coordinates, and they never own the text they describe. Because they are derived from a particular Document state, tokens are recomputable and must not be treated as permanent metadata: when the Document changes, the relevant tokens become stale and must be regenerated.

Tokens record *what lexical regions exist*. The next stage asks a different question:

> What semantic highlighting category should each region receive?

That distinction leads to **highlight classes**, which provide the boundary between syntax understanding and visual styling: **Section 32.5, From Tokens to Highlight Classes**.
