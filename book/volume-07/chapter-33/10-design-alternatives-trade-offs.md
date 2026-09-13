## 33.14 Design Alternatives and Trade-offs

There is no single way to configure an editor. Different approaches provide different balances between simplicity, flexibility, and complexity.

### Hard-Coded Settings

The simplest approach is to place settings directly in the source code:

```text id="h9v8qf"
Code
  │
  ▼
Behavior
```

For example, the editor could always use a tab width of `4` and always assign `Ctrl-S` to Save.

This is easy to implement and easy to understand, but users cannot change the behavior without modifying and rebuilding the program.

**Advantage:** Maximum simplicity.
**Limitation:** Very inflexible.

---

### Configuration File Only

A configuration file separates user preferences from the program:

```text id="w5kq3x"
File
  │
  ▼
Configuration
  │
  ▼
Behavior
```

This allows preferences to persist between sessions and can be changed without recompiling the editor.

**Advantage:** Persistent and user-friendly.
**Limitation:** Temporary changes require modifying the configuration file.

---

### Command-Line Configuration

Command-line options provide temporary overrides:

```text id="j4h7qk"
User Configuration
       │
       ▼
Command-Line Override
       │
       ▼
Final Configuration
```

For example, a user might temporarily select a different tab width when launching the editor.

This is particularly useful for automation, scripts, or one-off sessions.

**Advantage:** Convenient temporary customization.
**Limitation:** Not practical for every preference or every interactive setting.

---

### Environment Variables

Environment variables can also provide configuration:

```text id="9qz1pb"
Environment
     │
     ▼
Configuration
```

They are useful for deployment environments and process-level settings, but they are less convenient for ordinary editor preferences. Users generally expect persistent editor preferences to live in an editor configuration file rather than in their shell environment.

**Advantage:** Useful for external environments and automation.
**Limitation:** Less suitable for normal interactive preferences.

For the baseline editor, environment-variable configuration is therefore unnecessary.

---

### Runtime Configuration

A more advanced editor could allow configuration to change while the editor is running:

```text id="x4g8ns"
Running Editor
      │
      ▼
Change Configuration
      │
      ▼
Reconfigure
      │
      ▼
Continue Running
```

This provides considerable flexibility. A user could change themes, key bindings, or display settings without restarting the editor.

However, it introduces substantially more complexity. Components must respond to configuration changes, derived state may need to be rebuilt, and the editor must define exactly when new settings take effect.

For this editor, that complexity is not justified by the baseline requirements.

---

### The Baseline Choice

The editor therefore uses a deliberately simple layered model:

```text id="0j6q9x"
Built-in Defaults
       ↓
User Configuration
       ↓
Optional Command-Line Overrides
       ↓
Final Configuration
```

This provides the important benefits of configuration while keeping the runtime model predictable.

The configuration is established and validated **before normal editing begins**. Once the Event Loop starts, modules consume the resulting configuration rather than dynamically changing it.

Runtime reconfiguration can be introduced later if a future version genuinely requires it.

### Engineering Insight

The goal of configuration is not to maximize flexibility. It is to provide **useful customization without making the architecture unnecessarily complex**.

For the baseline editor, defaults, persistent user configuration, and optional command-line overrides provide a strong balance between simplicity and usability.
