# Volume VIII — Software Engineering

# Chapter 37 — Portability

## Introduction

The editor was built against Linux and POSIX as its reference platform. That gave us a concrete environment to develop and test in, but it left a question unexamined: which parts of the editor are genuinely portable, and which are merely assumptions inherited from the platform we happened to choose?

A terminal editor depends directly on facilities provided by the operating system and the terminal—input, terminal modes, dimensions, file descriptors, and filesystem operations—so it is especially exposed to platform differences. Portability is the discipline of containing those differences: keeping the editor's core behavior stable while the mechanisms beneath it are free to vary. It does not require every platform to behave identically.

This chapter builds the model progressively:

```text
Motivation — why a terminal editor is sensitive to its platform
    ↓
Boundary — separating editor behavior from platform mechanisms
    ↓
Foundations — the C library, POSIX, and platform APIs
    ↓
Targets — Linux, macOS and BSD, Windows, and WSL
    ↓
Mechanism — conditional compilation at the boundary
    ↓
Reflection — the mistakes that break portability
```

We begin by explaining why portability matters and where the platform boundary lies. We then examine the layers beneath that boundary—the C standard library, POSIX, and platform-specific APIs—before walking through the project's targets: Linux, macOS and BSD, Windows, and WSL. We close by examining conditional compilation as the mechanism that selects implementations, and by reviewing the common mistakes that let platform details leak into the Core.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain why a terminal editor is particularly sensitive to platform differences.
- Distinguish portability from identical behavior.
- Define the platform boundary and classify responsibilities as platform-independent or platform-facing.
- Explain the relationship between the C standard library, POSIX, and platform-specific APIs.
- Describe why Linux is the reference platform and what should remain independent of it.
- Explain what macOS and BSD share with Linux through POSIX, and where they still differ.
- Describe how Windows differs from the POSIX model and how handles differ from file descriptors.
- Explain why WSL is an execution environment rather than a separate platform.
- Describe when conditional compilation is appropriate and where it belongs.
- Recognize the common mistakes that allow platform dependencies to spread.
- Explain how a contained platform boundary keeps the Core stable across platforms.
