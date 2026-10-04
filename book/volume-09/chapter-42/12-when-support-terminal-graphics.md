# 42.12 When Should Our Editor Support Terminal Graphics?

The preceding sections have shown that modern terminals can provide capabilities far beyond basic character output.

They can support:

```text
True Color
Mouse Input
Images
Graphics Protocols
Vendor-Specific Extensions
```

However, the existence of a capability does not mean that our editor should implement it.

An important engineering question is therefore:

> **When is an advanced terminal capability justified as part of our editor?**

The answer should be based on requirements and architectural value rather than on what the terminal happens to make possible.

## The Baseline Editor Has a Different Goal

The editor developed in this book is fundamentally a terminal text editor.

Its core responsibilities are:

```text
Edit Text
Navigate Text
Search Text
Undo / Redo
Save Files
Render the Editing Interface
```

These capabilities do not require terminal graphics.

The baseline rendering architecture is intentionally built around:

```text
Editor State
     ↓
Renderer
     ↓
Screen Buffer
     ↓
Terminal Interface
     ↓
Terminal
```

The Screen Buffer represents the cell-based visual frame, while the Renderer remains separate from the terminal-specific output mechanism.

This architecture is sufficient for the baseline editor.

## Required Versus Optional Capabilities

It is useful to distinguish capabilities by whether the editor depends on them.

| Capability                            | Baseline Editor             |
| ------------------------------------- | --------------------------- |
| Text rendering                        | **Required**                |
| ANSI terminal control                 | **Required**                |
| Terminal input                        | **Required**                |
| Terminal dimensions                   | **Required**                |
| 24-bit color                          | Optional future enhancement |
| Mouse input                           | Deferred                    |
| Images                                | Deferred                    |
| Kitty graphics                        | Deferred                    |
| General graphics capability detection | Future platform feature     |

The distinction is important.

A required capability belongs to the environment the baseline editor is designed to operate within.

An optional capability should enhance the editor without becoming necessary for its fundamental operation.

## Why Images Are Deferred

Images provide little value to the fundamental editing model established by the book.

A text editor must be able to represent:

```text
Document
   ↓
Logical Text
   ↓
Editable Lines
```

An image does not naturally belong in this representation.

Introducing images would instead require additional concepts:

```text
Image Resource
Placement
Visibility
Lifecycle
Graphics Protocol
Terminal Capability
```

It would also complicate the rendering model, as discussed earlier.

Therefore, images are better treated as a future presentation capability rather than as part of the baseline editor.

## Why Kitty Graphics Are Deferred

Kitty graphics provide a concrete terminal graphics protocol, but they also introduce a terminal-specific dependency.

Supporting Kitty would require the editor to consider:

```text
Kitty capability
       ↓
Protocol generation
       ↓
Graphics resources
       ↓
Rendering synchronization
       ↓
Fallback behavior
```

The feature would therefore increase both implementation complexity and portability cost.

Unless the editor has a concrete requirement for displaying graphical content, this additional complexity is not justified.

Kitty graphics are therefore deferred.

## Why Mouse Input Is Deferred

Mouse input is somewhat different because it can improve usability without changing the underlying text model.

Nevertheless, the baseline editor can perform all fundamental editing operations through keyboard input.

Mouse support would introduce additional work in:

```text
Mouse reporting
Coordinate mapping
Window hit testing
Viewport mapping
Selection behavior
Terminal state management
```

The SRS explicitly keeps mouse support outside the initial scope of the editor. Therefore, it should remain deferred unless the requirements are later expanded.

## True Color Is Different

True color deserves a different classification.

Color is still part of the terminal's ordinary cell-based presentation model.

It does not necessarily require a second graphical resource system.

Conceptually:

```text
Text
  ↓
Screen Cell
  ├── Character
  ├── Foreground Color
  └── Background Color
```

This fits naturally into the existing Screen Buffer model.

Therefore, supporting 24-bit color in the future would be substantially less disruptive to the architecture than introducing arbitrary images.

It remains optional because the baseline editor can function without it.

## Capability Detection

Capability detection is also not automatically required merely because optional features exist.

If the baseline implementation only depends on a small set of required terminal behavior, introducing a generalized capability-detection framework may add complexity without immediate value.

A future feature can introduce the capability detection it actually needs.

For example:

```text
Future Graphics Feature
          │
          ▼
Capability Detection
          │
      ┌───┴───┐
      ▼       ▼
 Supported  Unsupported
      │         │
      ▼         ▼
 Graphics    Fallback
```

This avoids designing a large abstraction before the requirements justify it.

## Requirements Should Drive Complexity

The architectural decisions throughout this book follow a consistent principle:

```text
Requirement
     ↓
Architecture
     ↓
Implementation
```

The process should not be reversed:

```text
Terminal has feature
     ↓
We should implement feature
     ↓
Architecture becomes more complicated
```

The fact that a terminal supports a capability is not sufficient justification for adding it to the editor.

Instead, the feature should answer a concrete user or product requirement.

## A Useful Decision Test

For a future terminal capability, we can ask:

```text
1. Does the editor require this capability?
        │
        ├── No → Defer
        │
        └── Yes
             ↓
2. Can the existing architecture support it?
             │
             ├── Yes → Extend existing boundary
             │
             └── No
                  ↓
3. Is the additional complexity justified?
                  │
                  ├── No → Defer
                  │
                  └── Yes → Design extension
```

This prevents optional terminal features from driving unnecessary architectural changes.

## The Baseline Decision

For the editor developed in this book, the decision is therefore:

```text
Text Rendering
      │
      ▼
Required

ANSI Terminal Control
      │
      ▼
Required

Advanced Terminal Features
      │
      ▼
Optional / Deferred
```

More specifically:

```text
24-bit Color       → Optional future enhancement
Mouse              → Deferred
Images             → Deferred
Kitty Graphics     → Deferred
Capability System  → Future platform feature
```

The baseline implementation should remain focused on the requirements already established for the editor.

## What Would Change This Decision?

The decision is not permanent.

A future version could legitimately introduce terminal graphics if a new requirement made them valuable.

For example, a future requirement might state:

```text
The editor must display image previews
inside the terminal editing environment.
```

That would change the architectural situation.

The new requirement would justify investigating:

```text
Graphics abstraction
Capability detection
Graphics resource management
Terminal protocols
Fallback behavior
Security limits
Testing across terminals
```

The architecture would then evolve because the **requirements changed**, not because the technology existed.

## Keeping the Core Stable

Even if graphics are eventually added, the Core should remain focused on the concepts that belong to the editor itself.

The desired boundary remains:

```text
Core
├── Document
├── Cursor
├── Viewport
├── Editor State
└── Commands

Platform / Presentation
├── Terminal Interface
├── Cell Rendering
└── Optional Graphics
```

Terminal graphics should therefore remain outside the Document model and outside the fundamental editing logic.

This preserves the architectural direction established throughout the book: platform-specific concerns should remain separated from the platform-independent Core.

## Final Decision for the Book's Implementation

The implementation volume should therefore **not** implement terminal graphics unless the requirements are explicitly expanded before implementation begins.

The baseline editor will implement the capabilities necessary for its core text-editing purpose.

Advanced terminal capabilities remain architectural extensions that the design can accommodate later.

This gives us a clear boundary:

```text
                OUR EDITOR
                     │
          ┌──────────┴──────────┐
          │                     │
       Required              Optional
          │                     │
   Text editing          24-bit color
   Terminal input        Mouse
   Cell rendering        Images
   ANSI control          Kitty graphics
                         Capability detection
```

The important engineering decision is therefore not:

> **"Can our terminal do this?"**

It is:

> **"Does our editor need this badly enough to justify the additional complexity?"**

For the baseline editor developed in this book, terminal graphics do not meet that threshold.

They remain a **future platform capability**, while the Core architecture remains centered on portable, predictable text editing.
