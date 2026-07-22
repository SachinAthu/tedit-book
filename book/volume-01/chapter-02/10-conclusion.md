# Summary

Summarize the key concepts:

- The CPU executes instructions.
- RAM stores program state and data.
- A running program is a process.
- `printf()` writes to standard output.
- The operating system delivers that output to a terminal.
- The terminal emulator renders characters using a font.
- UTF-8 allows modern terminals to display characters from many languages.

---

# Exercises

### Review Questions

1. What is the difference between a program and a process?
2. Why doesn't `printf()` draw directly on the screen?
3. What is the purpose of standard output?
4. Why do terminal emulators use monospaced fonts?
5. What problem does UTF-8 solve?

### Thought Experiment

Suppose your terminal emulator crashes while your program is still running.

- Is your program necessarily terminated?
- What part of the display pipeline has failed?

Explain your reasoning.

---

# Preview of the Next Chapter

Although we've seen that text travels through a terminal before reaching the screen, we haven't explored what a terminal actually is. In the next chapter, we'll trace the history of terminals from mechanical teletypes to modern pseudo-terminals (PTYs), introducing concepts such as TTYs, shells, sessions, file descriptors, and process groups. These components form the environment in which every terminal-based application—including our text editor—runs.

---

I think this chapter should be **around 25–35 pages**, making it more substantial than Chapter 1. It introduces the complete text-display pipeline while intentionally stopping just before POSIX, TTYs, and ANSI escape sequences, leaving those topics for Chapters 3 and 4 as planned. This sequencing matches the educational philosophy and progression defined in the Book Bible.
