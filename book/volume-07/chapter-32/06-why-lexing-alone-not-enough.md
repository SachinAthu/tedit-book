# 32.6 Why Lexing Alone Is Not Enough

Lexical analysis is powerful because many syntax-highlighting decisions can be made by examining source text from left to right.

Keywords look like keywords.

Numbers look like numbers.

Strings have recognizable delimiters.

Comments have recognizable delimiters.

For a large class of programming languages, this is enough to produce useful highlighting.

However, not every highlighting decision can be made from a token in isolation.

Consider:

```text
int calculate(int value) {
    return value + 10;
}
```

The lexer can identify both occurrences of:

```text
calculate
value
return
```

as identifiers or keywords.

But it cannot necessarily determine the complete semantic role of every identifier simply from its characters.

For example, `calculate` may be a function name, while `value` may be a parameter or variable.

This introduces an important distinction:

```text
Lexical question:
"What kind of token is this?"

Syntax question:
"What role does this token play here?"
```

Syntax highlighting sometimes needs the answer to the second question.

## Lexical Meaning Versus Context

An identifier is a good example.

Suppose the lexer encounters:

```text
calculate
```

On its own, this is simply an identifier-shaped sequence of characters.

Now consider where it appears:

```text
int calculate(int value)
```

The surrounding structure suggests that `calculate` is a function declaration.

Compare that with:

```text
calculate(value);
```

Here, the same identifier is being used as a function call.

And compare it again with:

```text
int calculate = 10;
```

Here, `calculate` is a variable.

The text itself has not changed.

Its surrounding context has.

Therefore, a high-quality syntax highlighter may need information beyond the lexical token:

```text
                ┌─────────────┐
Source text ───►│    Lexer    │
                └──────┬──────┘
                       │
                       ▼
                    Tokens
                       │
                       ▼
                Contextual analysis
                       │
                       ▼
                Highlight classes
```

The lexer provides the raw material.

A later analysis stage can interpret that material.

## Strings Demonstrate Another Boundary

Context also matters when determining whether characters should be interpreted as syntax at all.

Consider:

```text
return value;
```

The word `return` is a keyword.

Now consider:

```text
"return value;"
```

Here, the entire sequence is part of a string literal.

The word `return` should not receive keyword highlighting.

The lexer can handle this because it tracks lexical state.

Conceptually:

```text
NORMAL STATE
     │
     │ "
     ▼
STRING STATE
     │
     │ "
     ▼
NORMAL STATE
```

While in the string state, characters that normally have special meaning are interpreted as string contents.

The same principle applies to comments.

For example:

```text
// if (value > 10)
```

The text inside the comment may resemble language syntax, but it should normally remain a comment.

This demonstrates that some context is already required during lexing itself.

## Lexical State Is Not Full Parsing

It is useful to distinguish **lexical state** from **syntax parsing**.

Lexical state answers questions such as:

```text
Are we currently inside a string?

Are we currently inside a comment?

Does this quote terminate the current string?

Does this sequence begin a comment?
```

A parser answers more structural questions:

```text
Is this identifier a function?

Is this name a type?

Does this token belong to a declaration?

Which tokens form this expression?
```

The difference can be visualized as:

```text
                 Lexical Analysis
                        │
              recognizes token boundaries
                        │
                        ▼
                     Tokens
                        │
                 Syntax Analysis
                        │
              interprets relationships
                        │
                        ▼
               Semantic highlighting
```

The distinction matters because syntax highlighting does not automatically require a complete language parser.

A lightweight editor can often obtain useful results from a combination of lexical rules and limited contextual analysis.

## A Simple Contextual Example

Consider:

```text
if (count > 10) {
    print(count);
}
```

A lexer might produce:

```text
IF
PUNCTUATION
IDENTIFIER
OPERATOR
NUMBER
PUNCTUATION
PUNCTUATION
IDENTIFIER
PUNCTUATION
IDENTIFIER
PUNCTUATION
PUNCTUATION
```

This is enough to highlight:

```text
if        → KEYWORD
count     → IDENTIFIER
10        → NUMBER
print     → IDENTIFIER
```

But a richer highlighter might recognize:

```text
print     → FUNCTION
```

The difference comes from contextual information.

The identifier `print` has not changed lexically. Its role has been refined.

This suggests that the syntax subsystem can operate in layers rather than forcing the lexer to perform every task.

## Not Every Language Needs the Same Depth

Different languages require different levels of analysis.

A simple language may need only:

```text
Lexer
  ↓
Highlight Classes
```

A more complex language may benefit from:

```text
Lexer
  ↓
Tokens
  ↓
Lightweight Syntax Analysis
  ↓
Highlight Classes
```

An even more sophisticated implementation might eventually use:

```text
Lexer
  ↓
Parser
  ↓
Syntax Tree
  ↓
Semantic Information
  ↓
Highlight Classes
```

The architecture developed in this chapter should leave room for these possibilities without requiring the simplest implementation to contain all of them.

This is especially important for the editor developed in this book.

The baseline syntax-highlighting subsystem should remain understandable and lightweight. More advanced incremental parsing infrastructure belongs to later architectural work rather than being introduced prematurely here.

## When Lexing Is Sufficient

Many useful highlighting decisions can be made entirely from lexical information.

Examples include:

```text
keyword
number
string
comment
operator
punctuation
```

For example:

```text
const int limit = 100;
```

can be highlighted effectively by recognizing:

```text
const → KEYWORD
int   → KEYWORD
limit → IDENTIFIER
=     → OPERATOR
100   → NUMBER
```

No full parser is necessary for this result.

This is an important engineering principle:

> Use the simplest analysis that produces the required result.

A syntax highlighter does not need compiler-level semantic understanding merely to distinguish comments from keywords.

## When Context Becomes Useful

Context becomes valuable when the editor wants to distinguish categories that share the same lexical form.

For example:

```text
IDENTIFIER
```

might become:

```text
VARIABLE
FUNCTION
TYPE
PARAMETER
CONSTANT
```

depending on its surrounding syntax.

Another example is punctuation.

The character:

```text
*
```

could represent multiplication:

```text
a * b
```

or a pointer-related construct:

```text
int *ptr;
```

The lexer can identify `*` as an operator-like token, but understanding its precise role requires context.

Similarly, parentheses can occur in many different constructs:

```text
function()
if (condition)
(value)
```

The characters themselves are identical.

Their syntactic roles differ.

This is where contextual analysis can improve the quality of highlighting.

## Context Does Not Mean Semantic Analysis

There is another important boundary.

A syntax highlighter may use enough context to distinguish:

```text
FUNCTION
VARIABLE
TYPE
```

without attempting to understand the complete meaning of the program.

For example, it does not necessarily need to know whether:

```text
calculate(value);
```

will actually execute successfully.

It only needs enough structural information to determine how the source should be highlighted.

Therefore:

```text
Syntax highlighting
        ≠
Complete compiler analysis
```

A highlighter can be useful while knowing very little about program correctness.

This distinction keeps the subsystem focused.

## The Cost of More Context

More contextual analysis generally provides better highlighting, but it also increases implementation complexity.

A purely lexical highlighter might work like this:

```text
Document
   │
   ▼
 Lexer
   │
   ▼
Highlight classes
```

A contextual highlighter adds another stage:

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
Contextual analysis
   │
   ▼
Highlight classes
```

A parser-based system goes further:

```text
Document
   │
   ▼
 Lexer
   │
   ▼
 Parser
   │
   ▼
Syntax structure
   │
   ▼
Highlight classes
```

Each additional layer introduces more state, more code, more failure cases, and potentially more work after edits.

The goal is therefore not to maximize analysis.

The goal is to provide an appropriate level of analysis for the editor's requirements.

## A Progressive Architecture

A useful design allows the syntax subsystem to evolve progressively.

The initial implementation can support:

```text
Document
   ↓
Lexer
   ↓
Tokens
   ↓
Highlight Classes
```

Later, contextual rules can be added:

```text
Document
   ↓
Lexer
   ↓
Tokens
   ↓
Lightweight Syntax Analysis
   ↓
Highlight Classes
```

The rest of the editor does not need to change fundamentally.

The Renderer still receives visual information.

The Theme still maps semantic classes to styles.

The Document still owns the source text.

This is an example of why architectural boundaries matter more than the complexity of the first implementation.

## The Practical Boundary for This Editor

For the editor developed in this book, syntax highlighting should begin with a lightweight analysis model.

The subsystem should be capable of recognizing common lexical constructs and, where useful, applying limited contextual rules.

It should not attempt to become:

- a full compiler,
- a language server,
- a semantic type checker,
- a complete static-analysis engine,
- or a generalized incremental parsing framework.

Those systems solve broader problems.

The syntax highlighter has a narrower purpose:

> **Analyze enough structure to produce useful semantic highlighting for the current Document.**

This boundary keeps the feature appropriate to the architecture while leaving room for future evolution.

## From Context to Lightweight Parsing

We can now see why a second analysis stage can be useful.

The lexer gives us:

```text
tokens
```

but some highlighting decisions require relationships between those tokens.

For example:

```text
IDENTIFIER
```

may need to become:

```text
FUNCTION
```

because of where it appears in a declaration.

That requires some understanding of token relationships.

## Summary

Lexical analysis alone cannot always determine how text should be highlighted. Characters such as `int`, `(`, or `*` take on different meanings depending on whether they appear inside a comment, a string, a declaration, or an ordinary expression, so some contextual analysis is required. This does not mean the editor must build a full compiler or language server: highlighting needs only enough structure to classify regions, and more analysis always brings added complexity and cost. A progressive approach—starting with a lightweight lexer and adding limited context later—keeps the subsystem appropriate to its purpose while preserving the architectural boundaries that let it evolve.

That is the model for the next section, **Section 32.7, Lightweight Syntax Parsing**, which introduces a deliberately limited form of structural analysis that improves highlighting without turning the editor into a full compiler.
