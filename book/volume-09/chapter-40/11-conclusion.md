# Chapter Summary

The Language Server Protocol (LSP) is a way to make an editor language-aware without embedding language intelligence. A separate **language server** process performs analysis, and the editor, acting as the **LSP client**, communicates with it through a standardized protocol:

```text
Editor (LSP Client) ── LSP ──► Language Server
```

Language intelligence therefore becomes an **optional integration layer**, not a core dependency. The editor keeps ownership of the Document, cursor, commands, rendering, and file management; the server maintains its own analysis state, kept synchronized with the editor's in-memory document. Communication is **asynchronous** — requests expect responses, notifications do not, and results can arrive later — so the event loop must never block while waiting for the server. Server failures must not invalidate the editor's authoritative Document.

The services built on the protocol — **diagnostics**, **completion**, and **hover** — follow one pattern: the editor provides document context, the server derives language information, and the editor presents it. Each introduces derived state, stale-result concerns, and latency trade-offs. LSP is distinct from syntax highlighting and from plugin architecture (Chapter 39), and it adds real costs: process lifecycle, protocol communication, synchronization, and trust.

The chapter's decision is deliberately conservative: LSP is an advanced extension that enhances the editor without redefining it. The core editor must remain a fully functional text editor with no language server — a self-contained baseline that later volumes can extend.

## Exercises

### Review Questions

1. Draw the communication flow between an editor and language server.
2. Explain why the language server should not own the editor's Document.
3. What happens if a diagnostic response arrives after the document has changed?
4. Why should completion requests not block the Event Loop?
5. Compare LSP with putting a C parser directly inside the editor.
6. Explain the difference between LSP and a plugin ABI.
7. Design a failure strategy for an unavailable language server.

## Preview of the Next Chapter

**Chapter 41 — Tree-sitter**

LSP delegates language intelligence to an external server, but that server still needs an efficient way to understand a document as it changes. The next chapter examines **Tree-sitter**, an incremental parsing library that builds and updates syntax trees as the user edits.

We will study how Tree-sitter performs incremental parsing, how syntax trees represent a document's structure, and how persistent trees enable fast, incremental analysis. We will also see how the editor can use syntax trees for **syntax highlighting** and structural navigation — the kind of language-aware behavior that complements the protocol-based services introduced in this chapter.
