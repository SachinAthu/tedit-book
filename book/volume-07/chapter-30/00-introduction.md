# Volume VII — Professional Features

# Chapter 30 — Search

## Introduction

A text editor that cannot search feels incomplete. Search appears simple—the user enters a query and the editor finds it—but a real editor must answer many more questions. Where does searching begin? What happens when there are many matches? How does the user move from one match to the next? What happens at the end of the Document?

Search is therefore a **system feature**, not a string-matching algorithm. It builds on the architecture established earlier in the book: the Document supplies the text, Search locates a match, the Cursor moves to it, the Viewport makes it visible, and the Renderer presents the result. Search works **with** these existing components without replacing them.

This chapter builds our understanding of search progressively:

```text
Literal — linear search
    ↓
Interactive — incremental search
    ↓
Expressive — regular expressions
```

We define the search problem precisely, examine the state an active search requires, and see how incremental search turns finding text into an interaction. We then compare regular expressions with literal matching, look at search from the user's point of view, and analyze its performance. Finally, we weigh design alternatives, study the bugs that commonly occur, and establish the deliberate scope of what we are not building.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain why search in an editor is a system feature rather than a single algorithm.
- Define the search problem: a directional, position-dependent operation that produces a logical match.
- Describe linear search and why it suits the baseline editor.
- Explain how searching works with a Document organized as logical lines.
- Identify the search state an active search requires and how it differs from the Cursor.
- Describe incremental search as an interaction model.
- Distinguish literal matching from regular-expression matching, and explain how regex changes the search problem.
- Identify the behaviors a predictable search user experience requires.
- Evaluate the performance trade-offs of the baseline search design.
- Compare search design alternatives and recognize common search bugs.
- State what the search feature deliberately does not build.