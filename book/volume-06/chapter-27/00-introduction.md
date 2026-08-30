# Volume VI — Algorithms & Data Structures

# Chapter 27 — Piece Table

## Introduction

Chapter 26 answered the cost of editing with the gap buffer, which keeps unused space near the cursor so that repeated local edits avoid moving existing text. That representation still treats the document as one mutable, contiguous sequence of characters.

The **piece table** takes a different approach. It keeps the underlying text in buffers and represents the current document as a sequence of references to portions of those buffers, so that editing changes the description of the document rather than the stored characters. The central question is:

> **How can a document be represented as stable text plus references, so that insertion and deletion modify a description instead of rearranging characters?**

We begin with the idea that motivates this alternative, then build the piece table from its two buffers and its piece descriptors. We examine how the document is reconstructed, how insertion and deletion work, how repeated editing changes the structure over time, and the invariants that keep it valid. We then see how logical positions map to stored text and analyze the complexity and trade-offs that determine when a piece table is worth choosing.

## Learning Objectives

Readers should be able to:

- explain why a piece table stores text in buffers and describes the document through references to ranges
- distinguish the Original Buffer, the Add Buffer, and the piece sequence
- explain why the Original Buffer is immutable and why the Add Buffer only grows
- describe a piece descriptor as a `(source, offset, length)` reference and explain why a piece is not a copy
- reconstruct the logical document by traversing the pieces in order
- explain how insertion appends to the Add Buffer and splits pieces without moving existing text
- explain how deletion removes or shortens pieces rather than erasing characters
- describe how repeated editing fragments the piece structure and lets the logical document and physical storage diverge
- state the invariants that define a valid piece table
- convert a logical document position into a source-buffer offset by locating the containing piece
- analyze the complexity of position lookup, insertion, deletion, traversal, and memory usage
- evaluate when a piece table's trade-offs make it a better choice than a simpler representation