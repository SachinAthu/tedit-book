# Volume VI — Algorithms & Data Structures

# Chapter 26 — Gap Buffer

## Introduction

Chapter 25 showed that inserting or deleting text in the middle of a contiguous array can require moving every element that follows. Interactive editing, however, rarely edits at random positions: users type in local bursts near the cursor.

This chapter studies a data structure designed around that workload—the **gap buffer**—and asks the central question:

> **How can unused space near the editing position reduce the cost of repeated local edits?**

We start with the idea that motivates the gap and the layout that realizes it, then establish the invariants that keep the buffer valid. We examine how the gap moves, how insertion and deletion consume or expand it, and how the buffer grows when space runs out. We then consider multiple lines, the role of editing locality and memory layout, and finally the failure modes that a careful implementation must avoid.

## Learning Objectives

Readers should be able to:

- explain why the gap buffer reserves unused space at the editing position
- describe the three logical regions of a gap buffer: text before, the gap, and text after
- state the gap-buffer invariant `0 ≤ gap_start ≤ gap_end ≤ capacity`
- distinguish the logical document from its physical storage
- explain how moving the gap changes the physical layout without changing the document
- explain why insertion and deletion at the gap cost `O(1)`
- describe why the gap must grow, why growth can cost `O(n)`, and why geometric growth keeps the amortized cost low
- compare one gap buffer for the whole document with a gap buffer per line
- relate the gap buffer's performance to the locality of editing and cursor movement
- describe the gap buffer's cache behavior and memory overhead
- identify common gap-buffer failure modes and how to debug them by checking invariants
