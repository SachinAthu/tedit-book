# Chapter Summary

Summarize the core ideas:

- Configuration separates defaults from user preferences.
- Every setting needs a valid default.
- Configuration should be loaded and validated before normal editing begins.
- Key bindings map input events to commands.
- Themes select presentation policies without modifying document contents.
- Tab width affects visual rendering, not logical document storage.
- Configuration should be centralized rather than parsed independently by modules.
- Invalid configuration should produce clear diagnostics.
- Configuration controls policy, not architecture.

## Exercises

### Review Questions

1. Why should configuration be separate from the Document?
2. What is the difference between a key event and a key binding?
3. Why should a TAB remain a TAB when the configured tab width changes?
4. Why should configuration be validated before entering the Event Loop?
5. Why should modules not independently read the configuration file?

### Thought Experiments

Suppose the user configures:

```text
tab_width = 8
```

while the editor previously used:

```text
tab_width = 4
```

Which parts of the system need to know about the change?

This should lead readers toward:

```text
Configuration
      ↓
Renderer
      ↓
Visual position
      ↓
Viewport
```

rather than changing the Document.

### Optional Challenge

Design a small configuration syntax supporting:

- `tab_width`
- `theme`
- `save`
- `quit`

without implementing the parser.

## Preview of the Next Chapter
