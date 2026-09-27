# 39.9 Security Considerations

A plugin system introduces a security boundary that does not exist in the same form for ordinary internal modules.

Plugins are external native code, and an in-process plugin generally executes with the same operating-system privileges as the editor.

Therefore:

> **A plugin must be treated as trusted code, not as untrusted data.**

## Plugin Loading

The editor should avoid blindly loading arbitrary files.

A plugin should at minimum pass the established checks:

```text
Discover
   ↓
Load
   ↓
Identify
   ↓
Validate ABI
   ↓
Initialize
```

An unknown or incompatible library should be rejected rather than executed.

## File and System Access

A plugin may potentially access resources available to the editor process, including:

```text
Files
Environment variables
Other processes
Network resources
User data
```

The plugin API should therefore expose only the capabilities that plugins actually need.

However, an in-process C plugin cannot be made a true security sandbox merely by restricting its API. Native code can potentially bypass those abstractions and access operating-system facilities directly.

## Plugin Trust

This creates a practical trust model:

```text
Official / trusted plugin
        ↓
Accept

Unknown / untrusted native code
        ↓
Do not treat as sandboxed
```

Users should therefore install plugins only from sources they trust.

## Stronger Isolation

If untrusted extensions must eventually be supported, the architecture would need stronger isolation, such as:

```text
Editor Process
      |
     IPC
      |
Plugin Process
      |
Sandbox / restricted privileges
```

That is substantially more complex than the baseline in-process plugin model.

## What is a IPC

**IPC** stands for **Inter-Process Communication**.

It is a way for **two separate processes** to exchange data and coordinate with each other.

In our plugin discussion, IPC becomes relevant when we put the plugin in a **separate process** instead of loading it directly into the editor.

### Without IPC

With an in-process plugin:

```text
+--------------------------------------+
|          Editor Process              |
|                                      |
|  Editor Core  <---->  Plugin        |
|       function calls                 |
|                                      |
+--------------------------------------+
```

The editor can directly call plugin functions because both are running in the same process.

For example:

```c
plugin_run();
```

The function call directly transfers execution to the plugin.

The problem is that both share the same address space. A serious plugin failure can therefore affect the editor.

---

### With IPC

With process isolation:

```text
+---------------------+        +---------------------+
|   Editor Process    |        |   Plugin Process    |
|                     |        |                     |
|   Editor Core       | <----> |      Plugin         |
|                     |  IPC   |                     |
+---------------------+        +---------------------+
```

They are **separate processes**.

The editor cannot simply do:

```c
plugin_run();
```

because `plugin_run()` exists in another process.

Instead, the editor sends a message:

```text
Editor
   |
   | "Run command X"
   ↓
IPC mechanism
   |
   ↓
Plugin
```

The plugin performs the operation and can send a response back:

```text
Plugin
   |
   | "Command completed"
   ↓
IPC mechanism
   |
   ↓
Editor
```

### Common IPC mechanisms

Operating systems provide several ways for processes to communicate:

```text
Pipes
Unix domain sockets
Shared memory
Signals
Message queues
```

For a plugin architecture, **Unix domain sockets** or **pipes** are examples of mechanisms that could be used.

---

### Why IPC improves isolation

Suppose the plugin crashes:

```text
+------------------+        +------------------+
|  Editor Process  |        | Plugin Process   |
|                  |        |                  |
|      Running     |        |     CRASH        |
+------------------+        +------------------+
```

The plugin's address space is separate from the editor's.

So, unlike an in-process plugin:

```text
Plugin crash
     ↓
Plugin process terminates
     ↓
Editor can potentially continue
```

This is why I mentioned IPC in **39.8 Plugin Failure and Isolation**.

But there is a trade-off:

```text
In-process
    ↓
Simple + fast
    ↓
Weak isolation

Separate process + IPC
    ↓
More complex + communication overhead
    ↓
Stronger isolation
```

For **our editor**, IPC is **not required for the baseline plugin architecture**. It is a possible future design if we ever want to support untrusted or strongly isolated plugins.

A useful way to remember it is:

> **Function calls communicate within a process; IPC communicates between processes.**

## Summary

The security boundary should be stated clearly:

> **The plugin interface controls architectural coupling; it does not make native plugin code trustworthy or sandboxed.**

For the editor's baseline architecture, the appropriate security strategy is therefore **explicit trust, controlled loading, minimal exposed capabilities, and clear documentation of the limits of in-process plugins**.

The next section brings these principles together into a concrete design: **39.10 A Minimal Plugin Architecture**.
