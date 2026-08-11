# Volume III — Core Editor Engine

# Chapter 13 — Document Model

## Introduction

Every text editor must answer one fundamental question:

> **How should a document exist in memory?**

It is easy to assume that a text file is "just a string"—a sequence of characters that the editor reads, modifies, and writes back. In reality, professional editors use specialized data structures because editing text efficiently is surprisingly difficult. Inserting a single character can require shifting thousands of others, splitting a line must be cheap enough to happen on every keystroke, and loading a file must be fast enough to feel instantaneous.

This chapter explores that problem from the ground up. We begin by distinguishing a **file**—the permanent representation of text stored on disk—from a **document**—the editable, in-memory model the editor actually works with. We then see why editors modify the document in memory rather than editing the file directly, and we define the responsibilities of the Document component: it owns the text and nothing else.

With those foundations in place, we ask what operations a document must support, and use those requirements to compare several ways text can be represented in memory: a single string, a vector of lines, a gap buffer, a piece table, and a rope. Each representation makes different trade-offs between simplicity, performance, memory usage, and implementation complexity. Finally, we explain why our editor deliberately begins with a vector of lines, and how the Document fits into the larger Editor State architecture.

This chapter establishes the conceptual model that later chapters (Cursor Engine, Command Engine, File I/O, and the data structure chapters in Volume VI) will build upon. By the end, you will understand not only how our editor stores text, but why that choice is an architectural decision rather than a mere coding detail.

## Learning Objectives

By the end of this chapter, readers should be able to:

- Explain why the document is stored in memory instead of edited directly on disk.
- Describe the responsibilities of the Document component.
- Understand what operations a document must support.
- Compare common document representations.
- Explain why our editor begins with a vector of lines.
- Recognize the trade-offs between simplicity and performance.
