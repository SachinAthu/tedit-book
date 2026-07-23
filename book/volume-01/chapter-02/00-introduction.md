# Volume I — Computing Foundations

# Chapter 2 — How Computers Display Text

## Introduction

Most programmers write:

```c
printf("Hello, World!\n");
```

and immediately see the text appear on the screen.

It feels almost magical.

However, between calling `printf()` and seeing the word **Hello** displayed, many different hardware and software components work together. The CPU executes instructions, memory stores data, the operating system manages the process, the terminal emulator interprets incoming bytes, and finally the graphics system renders characters onto the display.

Understanding this journey is essential because a terminal text editor relies entirely on this pipeline. Our editor will never draw pixels directly—it will communicate with the terminal by sending text and control sequences.

In this chapter, we'll examine every major component involved in displaying text and establish the foundation for the chapters that follow.

---

# Learning Objectives

By the end of this chapter, you should be able to:

- Explain the roles of the CPU and main memory.
- Describe what a process is.
- Distinguish between standard input and standard output.
- Explain how a terminal emulator differs from a console.
- Describe how characters are represented and rendered.
- Understand the basics of UTF-8 encoding.
- Trace the complete path from `printf()` to visible text on the screen.
