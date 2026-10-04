# 42.13 Architectural Lessons

Terminal graphics and other advanced terminal capabilities reveal several broader principles about terminal application architecture.

The important lessons are not specific to Kitty, images, or mouse input. They apply whenever an application operates through a terminal whose capabilities extend beyond basic character output.

## 1. A Terminal Is a Protocol Endpoint, Not a Framebuffer

The editor does not directly manipulate a physical display.

Its output passes through a protocol boundary:

```text
Editor
  │
  ▼
Terminal Interface
  │
  ▼
PTY
  │
  ▼
Terminal Emulator
  │
  ▼
Display
```

The terminal emulator interprets the protocols and determines how the resulting output is presented.

This explains why terminal graphics are fundamentally different from drawing directly into a framebuffer.

The editor communicates **what should be represented**, while the terminal emulator performs the actual display operation.

## 2. Terminal Capabilities Are Not Universal

Terminal capabilities vary between environments.

A feature may be:

```text
Supported
Unsupported
Partially supported
Implemented through another protocol
```

Therefore, advanced capabilities cannot automatically be treated as universal properties of the terminal environment.

This leads to a general rule:

> **Optional terminal capabilities must be treated as environmental capabilities, not assumptions of the application.**

The baseline editor avoids this problem by depending only on the terminal behavior required by its core design.

## 3. Graphics Introduce Resources Beyond Screen Cells

The Screen Buffer provides a bounded representation of ordinary cell-based output.

Graphics introduce another category:

```text
Screen Cells
    +
Graphics Resources
```

A graphical resource may have its own:

- identity,
- data,
- placement,
- lifetime,
- and cleanup requirements.

Consequently, a graphics-capable renderer may need to manage state that cannot be represented entirely by the ordinary Screen Buffer.

This does not invalidate the Screen Buffer design. It establishes its boundary:

> **The Screen Buffer represents cell-based presentation; it is not a general representation of every possible terminal visual resource.**

## 4. Input and Rendering Protocols Must Remain Separate from Editor Logic

Mouse input demonstrated the input side of this principle:

```text
Terminal Protocol
       ↓
Input Parser
       ↓
Abstract Event
       ↓
Editor Command
```

Terminal graphics demonstrate the output side:

```text
Editor State
       ↓
Renderer
       ↓
Terminal Interface
       ↓
Terminal Protocol
```

In both cases, protocol-specific representations are translated at architectural boundaries.

The Core should not need to know whether an input event originated from a particular terminal encoding or whether graphical output uses a particular vendor protocol.

## 5. Optional Capabilities Require Explicit Handling

Advanced features should not silently become dependencies.

The general model is:

```text
Optional Feature
      │
      ▼
Capability Available?
     / \
   yes  no
    │    │
    ▼    ▼
 enable fallback
```

The fallback does not necessarily need to reproduce the complete feature.

Its purpose is to preserve the editor's essential functionality.

For our editor, the fundamental requirement remains text editing. Advanced presentation or input capabilities must not prevent that functionality from operating.

## 6. Vendor-Specific Protocols Belong Behind Platform Boundaries

Kitty graphics demonstrate the cost of depending on a vendor-specific protocol.

The undesirable architecture would allow the protocol to spread through the application:

```text
Document
   ↓
Editor State
   ↓
Commands
   ↓
Kitty-specific behavior
```

The preferred architecture is:

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
Vendor-specific protocol
```

This keeps protocol-specific decisions near the platform boundary.

A future protocol can then be added without requiring the Core to understand its details.

## 7. The Core Should Remain Independent of Terminal Graphics

The Core represents the editor's fundamental concepts:

```text
Document
Cursor
Viewport
Editor State
Commands
```

None of these concepts fundamentally requires terminal graphics.

An image displayed by a terminal does not change what a document is.

A Kitty graphics sequence does not change what a cursor is.

A terminal-specific protocol does not change what an editing command means.

Therefore:

> **Terminal graphics should remain a presentation and platform concern rather than becoming a Core concern.**

This preserves the dependency direction established by the architecture.

## 8. Do Not Complicate the Architecture for Optional Features

The most important lesson is perhaps the simplest.

A terminal may provide capabilities that our editor does not need.

Adding every available capability would produce an architecture such as:

```text
Core
  │
  ├── Text
  ├── Mouse
  ├── Images
  ├── Graphics
  ├── Terminal-specific protocols
  └── Other optional features
```

This would increase coupling and make the Core harder to reason about.

The preferred design is:

```text
Core
  │
  ▼
Stable editor concepts
  │
  ▼
Platform boundary
  │
  ├── Required terminal behavior
  └── Optional capabilities
```

Complexity should therefore be introduced when requirements justify it, not merely because the underlying technology permits it.

## Summary

Chapter 42 ultimately reinforces a broader architectural principle used throughout this book:

> **A good architecture does not attempt to model every capability of its environment. It models the capabilities required by the application and isolates optional environmental features behind stable boundaries.**

For our editor, this means that terminal graphics, mouse input, and other advanced terminal extensions can remain future capabilities without weakening the baseline design.

The architecture is therefore intentionally conservative:

```text
                Editor Core
                    │
             stable concepts
                    │
                    ▼
                 Renderer
                    │
                    ▼
            Terminal Interface
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   Required terminal     Optional terminal
      behavior              capabilities
                              │
                     ┌────────┼────────┐
                     ▼        ▼        ▼
                   Color    Mouse   Graphics
```

The goal is not to build the most feature-rich terminal application possible.

The goal is to build an editor whose architecture remains **clear, predictable, portable, and extensible**, while providing well-defined boundaries for capabilities that may be added when future requirements justify their cost.
