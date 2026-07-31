# Volume I — Computing Foundations

# Chapter 5 – Raw Terminal Mode

## Introduction

The terminal is more than a display—it is also responsible for processing keyboard input before your program ever sees it. By default, the terminal provides a user-friendly interface designed for shells and command-line programs. A text editor, however, needs direct access to every key the user presses. This chapter explores how to temporarily disable the terminal's default behavior and place it into **raw mode**, giving our editor complete control over keyboard input.

## Learning Objectives

By the end of this chapter, readers should be able to:

- Explain why terminals preprocess keyboard input.
- Distinguish canonical mode from raw mode.
- Understand the role of the `termios` API.
- Describe how echo, signals, and control characters are handled.
- Explain why terminal settings must always be restored.
- Understand how raw mode fits into the editor's architecture.
