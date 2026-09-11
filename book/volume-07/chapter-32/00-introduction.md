# Volume VII — Professional Features

# Chapter 32 — Syntax Highlighting

## Introduction

A terminal text editor works with text: to a Document, `int count = 42;` is nothing more than a sequence of characters. Yet a modern editor displays that same text as keywords, identifiers, operators, and numbers. It does so without changing the document—it *understands* the text well enough to present it more clearly. That is syntax highlighting.

Syntax highlighting looks like a matter of color, but it is really an information-processing problem. The editor must analyze the text, decide what different regions represent, and hand that semantic information to the Renderer, which decides how each region looks. Getting this right requires a clear boundary: the Document stays authoritative, the syntax subsystem derives structure from it, a Theme maps that structure to styles, and the Renderer draws the result.

This chapter builds that pipeline progressively:

```text
Problem — why abstract structure from plain text?
    ↓
Pipeline — analysis stages and their boundaries
    ↓
Lexing — recognizing tokens and lexical state
    ↓
Parsing — adding context beyond tokens
    ↓
Presentation — classes, themes, and the renderer
    ↓
Integration — language detection, edits, performance
```

We begin with why syntax highlighting exists and why it must remain derived information. We then walk through the analysis pipeline, detailing the Lexer, tokens, and highlight classes before examining why lexical analysis alone is insufficient and how a lightweight parser adds context. Next we see how highlighting is applied to the Document and connected to the text buffer, how Themes map semantic classes to visual styles, and how the editor decides which language rules apply. We close with the practical concerns of updating highlighting after edits, performance and scalability, and the design alternatives and trade-offs involved.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain why syntax highlighting treats text as derived information rather than modifying the Document.
- Describe the syntax-highlighting pipeline and the responsibility of each stage.
- Explain how a Lexer recognizes tokens and tracks lexical state.
- Distinguish a token from a highlight class, and a token type from a semantic category.
- Explain why lexical analysis alone is insufficient and what a lightweight parser adds.
- Describe how highlighting information is associated with the Document and layered around the text buffer.
- Explain how Themes map highlight classes to visual styles without coupling to language analysis.
- Describe how the editor selects a language or highlighting mode for a Document.
- Explain how highlighting is (re)built incrementally after edits, and why it can always be discarded and regenerated.
- Evaluate the performance and scalability considerations of the baseline design.
- Compare the main implementation alternatives and their trade-offs.