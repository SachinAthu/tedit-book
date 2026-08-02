# Volume II — Software Architecture

# Chapter 7 — Software Requirements Specification

## Introduction

By the end of Volume I, we learned how to communicate with the terminal. We can enable Raw Mode, receive keyboard input immediately, and interpret everything from ordinary characters to multi-byte escape sequences. We now understand _how_ the terminal and our program exchange information.

But knowing how to receive input does not tell us how to build a text editor.

Where should we begin? Which modules should we design first? What features are essential, and which should wait? Without answering these questions, implementation becomes a series of disconnected decisions rather than a coherent design. Professional software projects avoid this by planning before they code.

This chapter introduces the **Software Requirements Specification (SRS)**—a structured document that describes what a system must accomplish before any implementation begins. We will define the scope of our editor, identify who will use it, write functional and non-functional requirements, and establish the constraints, assumptions, and success criteria that will guide every architectural decision throughout the remainder of this book.

This is the first chapter of Volume II. In the chapters that follow, we will transform these requirements into a concrete software architecture—dividing the editor into modules, assigning responsibilities, and defining how those modules collaborate to build a complete terminal text editor.

## Learning Objectives

By the end of this chapter, readers should be able to:

- Explain why defining requirements before writing code reduces project risk and prevents feature creep.
- Describe the purpose of a Software Requirements Specification and distinguish between _what_ software does and _how_ it is implemented.
- Define project scope by identifying both the features that are included and those that are intentionally excluded.
- Identify stakeholders and write requirements from the user's perspective rather than the developer's.
- Write functional requirements using clear terminology such as _shall_, _should_, and _may_, and organize them into logical categories.
- Distinguish between functional requirements, which describe software behavior, and non-functional requirements, which describe quality attributes such as performance, reliability, and maintainability.
- Recognize how constraints such as C99, POSIX, and single-threaded execution shape the design space by eliminating incompatible solutions.
- Document the assumptions an application makes about its operating environment and explain how those assumptions simplify design.
- Construct use cases that connect individual requirements into realistic user workflows and reveal missing behaviors.
- Define measurable success criteria that provide an objective endpoint for development and determine when a project is complete.
