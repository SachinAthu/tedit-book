# Volume I — Computing Foundations

# Chapter 6 — Keyboard Input

## Introduction

Now that we've enabled raw mode, every key press reaches our program immediately. However, receiving bytes is only the first step. A text editor must determine _what those bytes represent_. Some keys correspond to a single byte, while others produce sequences of multiple bytes. Understanding this encoding is essential before we can design an input handling system.

## Learning Objectives

By the end of this chapter, readers should be able to:

- Explain how keyboards generate input.
- Distinguish between characters and key events.
- Understand the role of ASCII in keyboard input.
- Recognize why UTF-8 characters may occupy multiple bytes.
- Explain how special keys are represented using escape sequences.
- Understand modifier keys such as Ctrl, Alt, and Shift.
- Design a strategy for parsing keyboard input.
- Appreciate why input parsing is separate from editor commands.
