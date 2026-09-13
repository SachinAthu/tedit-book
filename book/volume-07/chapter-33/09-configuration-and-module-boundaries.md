# 33.13 Configuration and Module Boundaries

Configuration should have **one clear entry point** into the editor. Individual modules should not read or parse the configuration file themselves.

The question is:

> Should every module read the configuration file itself?

**No.**

Configuration is loaded, parsed, and validated during startup. The resulting `Configuration` becomes part of `Editor State`, where other components can consume the values they need.

```text
                Configuration
                      │
                      ▼
                Editor State
                 /    |    \
                /     |     \
               ▼      ▼      ▼
            Input   Renderer  Commands
```

The Input Engine can consume key bindings, the Renderer can consume theme and display settings, and command-related code can consume relevant editing settings. None of them needs to know where the configuration came from or how it was parsed.

This keeps responsibilities separated:

```text
Configuration Layer
    └── Load, parse, validate

Editor State
    └── Own configuration

Modules
    └── Consume configuration
```

A design where every module independently reads the configuration file would duplicate parsing logic, create unnecessary dependencies, and make configuration behavior harder to reason about.

The better principle is simple:

> **Load configuration once, validate it once, and let modules consume the resulting state.**

This preserves the architecture established throughout the editor: **each component has a clear responsibility, and dependencies flow through well-defined boundaries.**
