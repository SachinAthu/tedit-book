# Volume VI — Algorithms & Data Structures

# Chapter 25 — Arrays and Strings

## Introduction

An editor is, at its heart, a machine for manipulating sequences of text. This chapter studies the simplest and most fundamental way to store such a sequence: the **array**.

Building on the complexity vocabulary from Chapter 24, it asks the central question:

> **How well does a contiguous sequence of characters support the operations a text editor performs?**

The answer reveals the chapter's key trade-off: an array's contiguous storage makes indexed access direct, but inserting or deleting a character in the middle of a line requires moving the elements that follow it.

We start with indexed access and fixed-size arrays, then build a **dynamic array** with explicit `length` and `capacity`, study **insertion** and **deletion**, and apply the same ideas to text as **strings** and dynamic strings. The chapter closes by comparing this baseline with alternatives—fixed and dynamic arrays, linked lists, Gap Buffers, Piece Tables, and Ropes—and justifying why our editor stays with a vector of lines using contiguous character storage.

## Learning Objectives

Readers should be able to:

- explain why an array is a natural representation for a line of text
- state the two properties of an array that make indexed access direct: contiguous storage and index-based addressing
- compute the address of an element from a base address, an index, and an element size
- distinguish **length** from **capacity**
- describe the limitation of fixed-size arrays for editable text
- identify the three components of a dynamic array: `data`, `length`, and `capacity`
- explain how a dynamic array grows and why growing may require copying the existing elements
- explain why growing by larger increments reduces the amortized cost of insertions
- trace a middle insertion and a middle deletion step by step, including the required shift direction
- explain why insertion and deletion costs depend on the position of the operation
- describe a C string's null-terminated byte representation and what that terminator costs
- explain why the editor's Document stores decoded Unicode code points rather than C strings
- describe a dynamic string in terms of `data`, `length`, and `capacity`
- compare fixed arrays, dynamic arrays, linked lists, Gap Buffers, Piece Tables, and Ropes as text representations
- justify the editor's chosen representation: a vector of lines with contiguous character storage