# Volume II — Software Architecture

# Chapter 8 — System Architecture

## Introduction

Chapter 7 defined what our editor must accomplish. We now have a Software Requirements Specification that describes every feature, constraint, and success criterion the completed editor must satisfy.

But a requirements document does not produce working software. It tells us the destination; it does not provide the map.

This chapter introduces the **system architecture** of our terminal text editor — the blueprint that transforms a list of requirements into a workable design. We will partition the editor into distinct components, assign each one a clear responsibility, define the interfaces through which they communicate, and trace the path data follows from a key press to a refreshed display.

Architecture is not an abstract exercise reserved for large projects. Even a small editor benefits enormously from deliberate organization. Without it, code becomes difficult to reason about, bugs multiply as modules accidentally interfere with one another, and every new feature requires rewriting existing code. With a well-defined architecture, the editor remains comprehensible, maintainable, and extensible as it grows throughout the remaining chapters of this book.

## Learning Objectives

By the end of this chapter, readers should be able to:

- Explain the purpose of software architecture.
- Understand why modular systems are easier to develop and maintain.
- Identify the major components of the editor.
- Understand the responsibilities of each component.
- Explain how data flows through the editor.
- Understand ownership of editor state.
- Read and reason about architectural diagrams.
