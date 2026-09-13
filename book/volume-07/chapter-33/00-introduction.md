# Volume VII — Professional Features

# Chapter 33 — Configuration

## Introduction

So far, our editor's behavior has been determined by the program itself. TAB always occupies the same width, `Ctrl-S` always saves, and the theme never changes. Each of these decisions is a **policy** — a preference that different users could reasonably want to change without altering how the editor works underneath.

Configuration separates those policies from the architecture. It lets the same editor support different tab widths, key bindings, and themes for different users while the Document, Cursor, Renderer, and Command Engine remain unchanged. Done well, configuration is a constrained layer of validated data: what the user selects, not how the editor is built.

This chapter builds that layer progressively:

```text
Scope — which decisions deserve to be configurable?
    ↓
Model — how configuration is represented and stored
    ↓
Defaults — every setting needs a valid baseline value
    ↓
Sources — user configuration and command-line overrides, with precedence
    ↓
File — persisting preferences between sessions
    ↓
Bindings — mapping input events to commands
    ↓
Validation — turning untrusted input into Editor State
```

We begin by defining why configuration matters and which decisions should be configurable at all. We then design the configuration model, its defaults, and how multiple sources combine under a clear precedence rule. Next we design the persistent configuration file, apply the model to key bindings, and define validation so invalid values never reach the editor — including why every module consumes one validated configuration rather than parsing the file itself. We close with the design alternatives and trade-offs, and the common mistakes to avoid.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain the distinction between architecture and policy, and why configuration controls policy.
- Decide which decisions should be configurable and which must remain internal implementation details.
- Describe a configuration model that stores data rather than behavior.
- Explain why every setting must have a valid, conservative default.
- Describe the precedence between defaults, user configuration, and command-line overrides.
- Explain how a configuration file serves as a persistence mechanism for policy.
- Describe how key bindings map input events to commands without changing command semantics.
- Explain how type, range, enumeration, and cross-setting validation keep invalid configuration out of Editor State.
- Explain why configuration must have one entry point rather than being parsed by every module.
- Evaluate the main configuration approaches and their trade-offs.
- Recognize common configuration mistakes and how to avoid them.
