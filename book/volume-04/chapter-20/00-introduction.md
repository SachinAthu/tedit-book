# Volume IV — Rendering Engine

# Chapter 20 — Scrolling

## Introduction

A terminal window can display only a small portion of a document at a time. As the cursor moves beyond the visible area, the editor must change which part of the document is shown.

This is the responsibility of the **Viewport**.

The Viewport tracks the visible region of the document using offsets. Scrolling adjusts those offsets so that the cursor remains visible while the Renderer displays the resulting region.

In this chapter, we will examine how vertical and horizontal scrolling work and how the Viewport interacts with the Cursor and Renderer.

## Learning Objectives

By the end of the chapter, the reader should understand:

- why a document needs a viewport
- the difference between document coordinates and screen coordinates
- how vertical scrolling works
- how horizontal scrolling works
- how viewport offsets represent the visible region
- how cursor movement interacts with the viewport
- how the editor keeps the cursor visible
- where scrolling logic belongs architecturally
- the complexity and trade-offs of the chosen model
