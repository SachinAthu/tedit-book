# 42.4 Terminal Capability Detection

A terminal application cannot safely assume that every terminal provides the same capabilities.

Basic terminal behavior is sufficiently widespread that an editor can depend on it within its supported environment. More advanced features are different. True color, mouse input, and graphical protocols may vary between terminal emulators and configurations.

This creates a fundamental problem:

> **Before using an optional terminal capability, the editor must determine whether that capability can be used safely.**

Capability detection is therefore the mechanism that connects the editor's desired presentation with the actual terminal environment.

## Why Capability Detection Is Necessary

Consider an editor that wants to display a particular feature:

```text
Editor wants feature
        │
        ▼
Does terminal support it?
      /     \
    yes      no
     │        │
   use it   fallback
```

Without this distinction, the editor may emit control sequences that the terminal does not understand.

The result could be:

- incorrect visual output,
- visible protocol data,
- unexpected terminal behavior,
- ignored commands,
- or a degraded user experience.

An optional feature should therefore never be treated as universally available simply because the editor knows how to request it.

## Capability Versus Terminal Identity

One possible approach is to identify the terminal emulator and assume that a particular version supports a known set of features.

For example:

```text
Terminal Name
      ↓
Known Terminal
      ↓
Assumed Capabilities
```

This can be useful, but it is not the same thing as directly establishing capability.

Terminal behavior can depend on:

- terminal emulator version,
- configuration,
- environment,
- transport layer,
- compatibility modes,
- and the capabilities exposed by the current environment.

Therefore, terminal identity should not automatically be treated as a complete capability description.

The architectural distinction is:

```text
Terminal identity
        ≠
Terminal capability
```

## Capability Information as Platform State

Capability information belongs near the platform boundary.

A conceptual representation might be:

```text
Terminal Capabilities
├── color support
├── true-color support
├── mouse support
├── graphics support
└── other optional features
```

The exact structure is an implementation decision for the later implementation volume. At the architectural level, the important point is ownership.

The Core should not contain terminal-specific capability checks throughout its editing logic.

Instead:

```text
Core
  │
  ▼
Renderer
  │
  ▼
Terminal Interface
  │
  ▼
Capability Information
```

This keeps platform knowledge localized.

## Capability Detection and Rendering

The renderer may have a preferred visual representation.

For example:

```text
Preferred presentation
        │
        ▼
True-color rendering
```

The terminal interface determines whether that representation can be emitted directly.

If it cannot, the renderer or terminal interface must select an appropriate fallback according to the architecture.

Conceptually:

```text
Visual Style
     │
     ▼
Terminal Capability
   /           \
supported    unsupported
   │              │
   ▼              ▼
preferred       fallback
encoding        encoding
```

The important principle is that capability detection should influence **presentation**, not the semantic meaning of the editor state.

A keyword remains a keyword even if the terminal cannot display its preferred color.

## Capability Detection and Input

The same principle applies to input capabilities.

Suppose mouse support is optional:

```text
Mouse capability
      │
      ▼
Can terminal provide mouse events?
      │
   ┌──┴──┐
  yes    no
   │      │
 enable   keyboard-only
```

The editor's core command model does not need to become dependent on mouse input.

If mouse events are available, they can enter the existing input pipeline as additional events:

```text
Terminal
   │
   ▼
Input Handler
   │
   ▼
Input Parser
   │
   ▼
Mouse Event
   │
   ▼
Input Engine
```

If they are unavailable, the existing keyboard-driven editor continues to function.

## Capability Detection and Graphics

Graphics make this issue even more important.

Unlike basic text rendering, graphical protocols are not necessarily shared across all terminal environments.

The editor may conceptually ask:

```text
Can this terminal display images
using the graphics mechanism required
by the editor?
```

The result may be:

```text
                 ┌── Supported ──► Graphics rendering
Capability ──────┤
                 └── Unsupported ─► Alternative rendering
```

This is why terminal graphics should be treated as an optional platform capability rather than a fundamental requirement of the editor.

## Detection Is Not the Same as Implementation

It is useful to separate two concerns.

**Capability detection** answers:

> What can the current terminal environment support?

**Capability implementation** answers:

> How does the editor use that capability?

These are related but distinct.

For example:

```text
Capability Detection
        │
        ▼
"True color available"
        │
        ▼
Rendering Policy
        │
        ▼
True-color Output
```

Similarly:

```text
Capability Detection
        │
        ▼
"Mouse available"
        │
        ▼
Input Configuration
        │
        ▼
Mouse Events
```

Keeping these responsibilities distinct prevents capability checks from being scattered throughout unrelated modules.

## Avoiding Capability Checks Everywhere

A poor architecture might contain code such as:

```text
if terminal_supports_true_color ...
if terminal_supports_mouse ...
if terminal_supports_graphics ...
```

throughout the editor.

This creates platform leakage.

A better model centralizes the knowledge:

```text
                 Terminal Interface
                        │
                Capability Information
                 /       |        \
                /        |         \
             Color     Mouse     Graphics
                │        │          │
                └────────┴──────────┘
                         │
                         ▼
                   Higher layers
```

Higher-level components consume the resulting capability information rather than repeatedly determining terminal behavior themselves.

## Graceful Degradation

Capability detection is valuable only when the editor knows what to do when a capability is unavailable.

For example:

```text
Preferred:
24-bit color
       │
       ▼
Unavailable?
       │
       ▼
Use supported color representation
```

Or:

```text
Preferred:
graphical image
       │
       ▼
Unavailable?
       │
       ▼
Use text representation or omit optional image
```

The exact fallback depends on the feature.

The general principle is:

> **Optional capabilities should degrade without compromising the core editor.**

## Capability Detection and the Baseline Editor

The baseline editor described by this book intentionally has a smaller scope than a modern graphical terminal application. The SRS defines the core editor around keyboard interaction, terminal rendering, file operations, cursor movement, scrolling, and related fundamental behavior, while advanced features such as mouse support are outside the initial implementation scope.

Therefore, capability detection should not become a reason to complicate the baseline implementation unnecessarily.

Instead, it provides an architectural mechanism for future extensions.

The editor can begin with:

```text
Required capabilities
        │
        ▼
Baseline terminal interface
```

and later extend this to:

```text
Required capabilities
        +
Optional capabilities
        │
        ▼
Capability-aware terminal interface
```

## A General Design Rule

The discussion can be summarized with a simple rule:

```text
Do not assume.
Detect capability.
Use capability when available.
Provide a fallback when practical.
Keep capability knowledge at the platform boundary.
```

This approach is particularly important for terminal graphics because graphical features tend to have greater variation between terminal environments than ordinary character-cell rendering.

The next section examines the first major capability that goes beyond the cell model: **images in terminals**.
