# Volume IX — Advanced Topics

# Chapter 39 — Plugin Architecture

## Introduction

The editor we have built is modular, but every module is part of the same program—compiled, linked, and released together. A plugin breaks that unity: it is independently developed code that crosses a defined boundary into the running editor. This changes the architectural problem from *how components inside the program communicate* to *how external code can extend the editor safely and predictably*.

This chapter builds that extension model progressively:

```text
Motivation — why plugins change the architecture
    ↓
Boundaries — what may cross between editor and plugin
    ↓
Interface — designing a deliberate plugin contract
    ↓
ABI — binary compatibility when source is not enough
    ↓
Versioning — validating compatibility before activation
    ↓
Dynamic Loading — how external code reaches the editor
    ↓
Safe Extension — preserving the editor's invariants
    ↓
Failure and Isolation — what happens when a plugin fails
    ↓
Security — trusting code that shares the editor's process
    ↓
Minimal Architecture — a small, versioned plugin boundary
    ↓
Alternatives — the trade-offs considered along the way
    ↓
Our Editor — what it would need to support plugins
```

We begin by explaining why a plugin is not merely another module and where its boundary should lie. We then design a plugin interface, examine ABI compatibility, versioning, and dynamic loading, and consider how to extend the editor safely despite plugin failure and security risk. We close by assembling these ideas into a minimal plugin architecture and by identifying exactly what our editor would need to support plugins.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain why a plugin changes the editor's architecture.
- Define the plugin boundary and describe what should not cross it.
- Design a plugin interface as a contract of capabilities.
- Distinguish API compatibility from ABI compatibility.
- Explain why plugin and interface versions are different.
- Describe how the editor loads, validates, and initializes a plugin.
- Explain how plugins can extend the editor without violating its invariants.
- Distinguish an extension boundary from a fault-isolation boundary.
- Describe the security limits of an in-process plugin system.
- Assemble these principles into a minimal plugin architecture.
- State what our editor would need to support plugins.