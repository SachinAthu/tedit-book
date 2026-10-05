# 43.5 Neovim — Refactoring the Editor Around APIs

Neovim provides a useful contrast with both Nano and Vim.

Nano demonstrates **simplicity**.

Vim demonstrates how a powerful **modal command model** can shape an editor.

Neovim adds another important idea:

> **An editor can be designed not only as an application, but also as a platform that other programs can communicate with.**

This changes the architectural boundaries considerably.

## From Editor to Platform

A traditional editor can be viewed primarily as:

```text
              Editor
                |
        +-------+-------+
        |               |
      Input          Rendering
        |               |
        +-------+-------+
                |
             Document
```

An API-oriented editor introduces another boundary:

```text
                 Editor Core
                      |
          +-----------+-----------+
          |                       |
      Terminal UI              API Layer
                                  |
                    +-------------+-------------+
                    |             |             |
                  Plugin        Client        Tool
```

The editor is no longer required to receive all commands exclusively from its own terminal interface.

Other software can interact with the editor through defined interfaces.

This is a major architectural shift.

## APIs Become Architectural Boundaries

An API is more than a convenient collection of functions.

It establishes a boundary between the editor's internal implementation and external consumers.

Conceptually:

```text
+-------------------------+
|       Editor Core       |
|                         |
|  Document               |
|  Cursor                 |
|  Commands               |
|  State                  |
+------------+------------+
             |
             | Public API
             v
+-------------------------+
| External Consumers      |
|                         |
| Plugins                 |
| Scripts                 |
| Tools                   |
| Clients                 |
+-------------------------+
```

The core can change internally while attempting to preserve the externally visible contract.

This introduces an important engineering concern:

> **Once functionality becomes part of an API, other software may depend on it.**

The API therefore becomes part of the architecture's stability boundary.

## Why This Changes the Design

Our editor can make relatively direct assumptions because its components are controlled together.

For example:

```text
Command Engine
      |
      v
Document
```

The Command Engine and Document are both internal components of the same application.

With an external API, the relationship becomes:

```text
External Code
      |
      v
Public API
      |
      v
Editor Core
      |
      v
Document
```

The public API must define what external code is allowed to do.

This raises questions that are much less important in our baseline editor:

- Which operations are exposed?
- What arguments do they accept?
- What state can external code access?
- Can external code modify documents?
- What happens when an API operation fails?
- How are API changes kept compatible?
- What happens when an extension crashes?

These are architectural questions rather than merely implementation details.

## Extensibility Moves Outward

Once an editor supports a substantial extension ecosystem, the core editor should not need to contain every possible feature.

Instead:

```text
                   Editor Core
                       |
              +--------+--------+
              |        |        |
              v        v        v
           Plugin    Plugin    Plugin
             A         B        C
```

Each extension can provide functionality that does not need to become part of the editor's permanent core.

This supports an important principle:

> **Keep the core responsible for fundamental editor behavior; move optional behavior behind extension boundaries.**

This idea connects directly with our earlier discussion of plugin architecture.

Our baseline editor intentionally does not require such an ecosystem. However, studying Neovim demonstrates what happens when extensibility becomes an important architectural requirement.

## Separation of Interface and Core

An API-oriented design also encourages us to distinguish the editor's core from the interface through which users or programs interact with it.

Conceptually:

```text
             +----------------+
             |   Editor Core  |
             +-------+--------+
                     |
              +------+------+
              |             |
              v             v
         Terminal UI      API
              |             |
              v             v
           Human        Software
```

This is a broader form of the separation we already established in our own architecture.

Our Renderer is separated from the core editor state:

```text
Editor State
     |
     v
 Renderer
     |
     v
Terminal
```

An API-oriented editor extends the same architectural thinking:

```text
                 Editor State
                      |
                +-----+-----+
                |           |
                v           v
             Renderer      API
                |           |
             Terminal    Extensions
```

The core does not have to know whether a requested operation originated from a human pressing a key or from another program invoking an API.

## More Interfaces Mean More Contracts

This flexibility comes with a cost.

Every public boundary creates a contract that must remain correct.

For example:

```text
                 Editor Core
                      |
             +--------+--------+
             |        |        |
          Terminal   API    Extension ABI
```

Each boundary introduces additional concerns:

- validation;
- error handling;
- compatibility;
- lifecycle management;
- versioning;
- security;
- testing.

This illustrates a recurring principle throughout the book:

> **Abstraction reduces coupling, but maintaining abstractions has a cost.**

An API is valuable when the flexibility it provides justifies that cost.

## Neovim and Our Editor

The comparison can be summarized like this:

```text
Our Editor

Human
  |
  v
Terminal
  |
  v
Editor
```

versus a more extensible architecture:

```text
                 Editor
                    |
        +-----------+-----------+
        |                       |
     Terminal                  API
        |                       |
      Human              Plugins / Tools
```

Our architecture deliberately stops closer to the first model.

That is not a limitation we accidentally inherited.

It is a consequence of the project's scope.

The SRS defines the baseline system as a small, single-document terminal editor and explicitly places advanced extensibility mechanisms outside the core implementation scope. 02-software-requirements

Neovim therefore teaches us an important lesson without implying that our editor should become Neovim:

> **When extensibility becomes a requirement, interfaces and APIs become architectural elements rather than optional conveniences.**

## Summary

Neovim demonstrates that an editor can be designed as a platform as well as an application. Once functionality is exposed through an API, that API becomes a stability boundary that other software depends on, and extensibility moves outward so the core need not contain every feature.

That flexibility has a cost: each public boundary creates a contract that must be validated, maintained, and kept compatible. The principle is the same one seen throughout the book—abstraction reduces coupling, but maintaining abstractions has a cost, and APIs become architectural elements when extensibility becomes a requirement.

The next section examines Emacs, which takes this principle further by making extensibility and programmability central to the editor itself.
