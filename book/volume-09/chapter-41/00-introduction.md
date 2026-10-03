# Volume IX — Advanced Topics

# Chapter 41 — Tree-sitter and Incremental Parsing

## Introduction

Chapter 40 examined how an editor obtains language intelligence from an external process. This chapter addresses a complementary problem: how the editor itself understands the **structure** of the source code it displays. A compiler parses a finished program once, but an editor works on source that changes with every keystroke. Maintaining a structural representation of that code requires a parser that can update its analysis incrementally. The central question is:

> **How can an editor maintain a structural representation of its source code that stays useful while the document changes continuously under the cursor?**

Tree-sitter is a concrete system built around this idea. It is an incremental parser that produces a syntax tree — a persistent, source-oriented structure that highlighting, folding, navigation, and other editor features can share.

We begin by explaining why incremental parsing matters and how a syntax tree differs from a token stream. We then define what "incremental" really means, introduce Tree-sitter as an incremental parser, and follow how a single edit is represented and propagated through the tree. We examine the structure and ranges of syntax-tree nodes, contrast concrete and abstract syntax trees, and see how error recovery keeps the parser useful while code is incomplete. From there we consider incremental syntax highlighting and the structural features a tree enables, relate Tree-sitter to the Language Server Protocol, and weigh complexity, performance, and design trade-offs before deciding why our baseline editor deliberately does not adopt Tree-sitter.

## Learning Objectives

Readers should be able to:

- explain why incremental parsing matters for an interactive editor and why rebuilding the entire tree after every edit is unnecessary
- contrast tokens with syntax trees and describe the structural information a tree adds
- explain what "incremental" really means, including why local changes can have non-local effects
- describe Tree-sitter as an incremental parsing component and define the boundary between it and the Document
- explain how an edit is represented and why the syntax tree must correspond to the current document version
- describe syntax-tree nodes, parent–child relationships, and source ranges
- contrast concrete and abstract syntax trees and explain why editors benefit from a source-oriented representation
- explain how error recovery lets a parser handle incomplete and syntactically invalid code
- describe how highlighting can be derived incrementally from a syntax tree
- identify structural editor features enabled by a syntax tree
- compare Tree-sitter with the Language Server Protocol and explain how the two coexist
- evaluate the complexity, performance, and design trade-offs of adopting Tree-sitter
- explain why the baseline editor deliberately does not adopt Tree-sitter
