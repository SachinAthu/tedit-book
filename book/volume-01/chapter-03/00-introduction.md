# Volume I — Computing Foundations

# Chapter 3 — The Unix Terminal

## Introduction

Open a terminal window and type a command. You see a prompt, your command, and its output—and it is tempting to think of the terminal as little more than a text window where you run programs.

That picture is far too simple.

The terminal is one of the oldest and most influential ideas in computing. Behind that ordinary-looking window is a layered system that has evolved over more than half a century: from the physical **teletypes** of the earliest mainframes, to the virtual terminals built into the Unix kernel, to the **terminal emulators**, **pseudo-terminals (PTYs)**, and **shells** that connect modern graphical desktops to the programs they run.

Every time you launch a program in the terminal, the operating system establishes a **terminal session**, connects that program to a set of **standard file descriptors**, and uses **process groups** to decide which program should receive your keyboard input. None of this is visible in the window itself, yet all of it determines how a terminal application—including the text editor we will build—actually behaves.

This chapter explains that environment from first principles. By the end, you will know exactly what your editor is connected to when it starts: where its input comes from, where its output goes, and how the operating system, the shell, and the terminal emulator cooperate to make a simple line of text appear on the screen.

## Learning Objectives

After completing this chapter, readers will be able to:

- Explain the historical evolution of Unix terminals.
- Distinguish between a terminal emulator, TTY, and PTY.
- Understand the role of the shell.
- Describe how a terminal session is established.
- Explain standard file descriptors.
- Understand foreground and background process groups.
- See how all these components work together.
