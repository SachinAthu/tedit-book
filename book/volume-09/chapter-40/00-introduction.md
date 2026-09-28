# Volume IX — Advanced Topics

# Chapter 40 — Language Server Protocol (LSP)

## Introduction

Chapter 39 examined how plugins extend an editor through a defined interface. The Language Server Protocol (LSP) solves a related problem from a different direction: instead of adding language-specific code to the editor, it places language intelligence in a separate process and standardizes the boundary across which the editor communicates with that process.

The editor acts as the **LSP client** and the external program acts as the **language server**. The protocol defines how the client requests language services — such as diagnostics, completion, and hover — and how the server provides them. The central question is:

> **How can an editor obtain language intelligence from an external process while keeping its core architecture independent of any particular programming language?**

We begin by examining why language intelligence belongs outside the editor, then define the protocol and the client–server architecture. We follow the language-server lifecycle and document synchronization, study diagnostics, completion, and hover as services built on the protocol, and examine how requests, notifications, and asynchronous results integrate with the event loop. We close by weighing the trade-offs that determine where LSP belongs in our editor.

## Learning Objectives

Readers should be able to:

- explain why language intelligence belongs outside the editor
- describe LSP as a protocol and distinguish the LSP client from the language server
- describe the LSP client as an adapter and explain why the editor keeps ownership of the Document
- explain how a language server is selected, started, initialized, and eventually shut down
- explain why the language server must be kept synchronized with the editor's in-memory document
- describe diagnostics as derived, asynchronous information that the editor presents but does not produce
- explain how completion turns language-server candidates into ordinary editing operations
- explain how hover provides contextual information without modifying the document
- contrast requests, notifications, and asynchronous results and explain why the event loop must not block
- evaluate the trade-offs that make LSP an optional advanced subsystem rather than a dependency of the core editor
