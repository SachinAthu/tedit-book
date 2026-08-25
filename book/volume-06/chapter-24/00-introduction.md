# Volume VI — Algorithms & Data Structures

# Chapter 24 — Complexity Analysis

## Introduction

In previous volumes, we built a working editor: the Document holds text, the Cursor navigates it, the Renderer draws it, and the File Manager persists it. Along the way we made many design decisions about how those components represent data.

This chapter steps back and asks the central question of this volume:

> **How much work does each editor operation require, and how does that work grow as the document grows?**

We develop a vocabulary and a method for answering that question—time, space, memory locality, and latency—so that in the chapters that follow we can meaningfully compare the data structures that can represent editable text.

## Learning Objectives

Readers should be able to:

- explain why correctness alone is not enough when evaluating an editor operation
- distinguish **time**, **space**, and **memory locality** as dimensions of cost
- identify the relevant input size for a given editor operation
- express the growth of an operation's cost using **Big-O notation**
- describe best case, average case, and worst case, and why worst case matters for an interactive editor
- identify what determines the time complexity of common editor operations
- analyze the additional memory an operation requires beyond the data it processes
- explain how cache locality affects practical performance beyond Big-O
- relate complexity to the **latency** a user experiences
- explain **amortized complexity** and how occasional expensive operations can still be efficient overall