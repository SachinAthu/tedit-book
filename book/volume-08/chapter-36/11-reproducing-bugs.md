# 36.11 Reproducing Bugs

A bug that cannot be reproduced is difficult to diagnose, difficult to fix confidently, and difficult to verify afterward.

For a stateful terminal editor, reproduction is especially important because behavior depends on a sequence of events:

```text id="j4g0lo"
Initial State
     │
     ▼
Input 1
     │
     ▼
State 1
     │
     ▼
Input 2
     │
     ▼
State 2
     │
     ▼
...
     │
     ▼
Failure
```

The visible failure is often the result of a particular sequence rather than a single operation.

A disciplined debugging process therefore begins by turning an observed failure into a **repeatable sequence of conditions and operations**.

The testing strategy established for the editor makes the same principle explicit: tests should be deterministic, and regression tests should preserve corrected defects at the lowest level that reproduces each failure.

## What Does Reproduction Mean?

To reproduce a bug means to recreate the conditions under which the same incorrect behavior occurs.

A useful reproduction description contains:

```text id="w0d5c2"
Initial State
     +
Environment
     +
Operation Sequence
     +
Observed Result
```

For example:

```text id="1j2c8d"
Initial:
    Document contains 20 lines
    Cursor at row 10

Operations:
    Move Down
    Move Down
    Delete line
    Move Down

Observed:
    Cursor moves outside the valid document range
```

This is much more useful than:

> The cursor sometimes becomes invalid.

The first description provides something that another developer can attempt to execute.

## Reproduction as a Sequence

Interactive editor bugs are often sequence-dependent.

Consider:

```text id="o8d5t6"
Open file
   ↓
Move down
   ↓
Delete line
   ↓
Move up
   ↓
Insert text
   ↓
Save
   ↓
Bug
```

Each operation changes the state on which the next operation operates.

Therefore, reproducing the bug requires preserving the relevant sequence.

This is particularly important for:

- cursor movement,
- scrolling,
- buffer switching,
- undo/redo,
- document modification,
- save behavior,
- configuration,
- terminal interaction.

The sequence is part of the bug's conditions.

## Start With the Smallest Known Reproduction

A long sequence may reproduce a problem:

```text id="r8y6v3"
A → B → C → D → E → F → G → H → I → BUG
```

But some of those operations may be irrelevant.

The goal is to reduce it:

```text id="m3x7fp"
C → D → G → BUG
```

and eventually:

```text id="0c8p8r"
D → G → BUG
```

This is called **minimizing the reproduction**.

The smaller the sequence, the easier it becomes to reason about the cause.

```text id="2q7k0h"
Large Reproduction
        │
        ▼
Remove irrelevant operation
        │
        ▼
Retest
        │
        ├── still fails → keep reduction
        │
        └── no longer fails → restore operation
        │
        ▼
Minimal Reproduction
```

A minimal reproduction is valuable because it reduces the number of possible causes.

## Example: A Cursor Bug

Suppose a developer observes:

> After several editing operations, pressing Down moves the cursor to an invalid row.

The initial reproduction might be:

```text id="f7qzom"
1. Open file
2. Move down
3. Move down
4. Move right
5. Delete character
6. Press Enter
7. Move up
8. Delete line
9. Move down
10. Move down
11. Bug
```

This is too broad.

Try removing operations:

```text id="tqksl4"
Open file
Delete line
Move down
Move down
```

If the bug still occurs, several operations can be discarded.

Eventually the developer may discover:

```text id="o2t3l9"
Open file
Delete line
Move down
Move down
```

is sufficient.

Now the investigation can focus on:

```text id="t6c7x2"
Delete line
     ↓
Document line count changes
     ↓
Cursor state
     ↓
Move down
```

The debugging search space is dramatically smaller.

## Determinism

A useful reproduction should behave consistently.

Suppose:

```text id="17y5gt"
Run 1 → failure
Run 2 → success
Run 3 → failure
Run 4 → success
```

The reproduction is not yet deterministic.

The developer should identify what additional condition affects the result.

Possible sources include:

- terminal dimensions,
- timing,
- input timing,
- filesystem state,
- uninitialized memory,
- random state,
- environment variables,
- previous application state.

The project's testing strategy explicitly requires deterministic tests: given the same initial state and operation sequence, the same result should occur every time.

Determinism is therefore not merely a testing concern. It is also a major debugging advantage.

## Recording the Initial State

The initial state is part of the reproduction.

For example:

```text id="k11m7f"
Initial Editor State

Buffer:
    example.txt

Document:
    50 lines

Cursor:
    row = 20
    column = 5

Viewport:
    row_offset = 15

Dirty:
    false
```

Without this information, another developer may reproduce the same operations but start from a different state.

For state-dependent bugs:

```text id="22v7wp"
Same commands
+
Different initial state
=
Different behavior
```

Therefore, a good reproduction describes the initial conditions explicitly.

## Reproduction and Test Fixtures

A test fixture provides a controlled initial state from which a test can execute.

For example:

```text id="0f4b1v"
Fixture
   │
   ├── Document with known lines
   ├── Cursor at known position
   ├── Viewport at known offset
   └── Known configuration
          │
          ▼
      Operation
          │
          ▼
       Assertion
```

This allows the same reproduction to be executed repeatedly.

For example:

```text id="p2n8zn"
Given:
    3-line document
    cursor at final line

When:
    delete current line

Then:
    cursor remains valid
```

If this sequence reproduces a defect, it can become a regression test.

This is exactly the direction encouraged by the project's testing strategy: preserve corrected defects at the lowest level that meaningfully reproduces the failure.

## Reproducing Input Bugs

Input bugs require preserving not only the logical commands but sometimes the actual input sequence.

For example:

```text id="2y7c0j"
ESC [ A
ESC [ A
Backspace
Enter
```

may produce a different result from a simplified sequence such as:

```text id="q4y1p9"
Arrow Up
Arrow Up
Backspace
Enter
```

if the problem occurs inside the Input Parser.

For parser bugs, the raw byte sequence can therefore be part of the reproduction.

```text id="0q7vnd"
Raw Bytes
    ↓
Input Parser
    ↓
Key Events
```

For higher-level command bugs, the raw bytes may no longer matter.

The appropriate level of reproduction depends on where the failure occurs.

## Reproducing Rendering Bugs

Rendering problems may depend on terminal dimensions or specific document contents.

For example:

```text id="8f7v2s"
Terminal:
    80 × 24

Document:
    long lines

Cursor:
    near right edge

Operation:
    move right

Observed:
    incorrect horizontal scrolling
```

Changing the terminal size to:

```text id="gr2m8w"
120 × 40
```

may make the problem disappear.

That does not necessarily mean the bug is fixed.

It may mean that the reproduction depends on a specific viewport geometry.

Therefore, the reproduction should record relevant terminal conditions.

This is particularly important for Viewport and rendering behavior because the visible frame depends on terminal dimensions.

## Reproducing File-System Bugs

File-related bugs can depend on external conditions.

For example:

```text id="t5v9z4"
Save failure
```

may depend on:

- file permissions,
- nonexistent directories,
- invalid paths,
- filesystem capacity,
- conflicting files.

A useful reproduction should describe the relevant external condition rather than simply saying:

> Saving failed.

For example:

```text id="q2n6se"
Initial:
    Document dirty

Environment:
    target path cannot be written

Operation:
    Save

Expected:
    Dirty Flag remains true
    Message Bar reports failure

Observed:
    Dirty Flag becomes false
```

This turns an environmental failure into a precise test condition.

## Reproducing Memory Errors

Memory errors can be especially difficult because the visible crash may depend on memory layout.

A useful reproduction should therefore include:

```text id="d3x0v7"
Input sequence
+
Document contents
+
Allocation conditions
+
Build configuration
+
Diagnostic configuration
```

For example:

```text id="6p2k4h"
Open document with long line
Insert characters repeatedly
Delete characters repeatedly
Switch buffer
Repeat
```

If a sanitizer reports an invalid access, the reproduction should be preserved so that the fix can be tested against the same sequence.

A memory bug that cannot be reproduced reliably is much harder to prove fixed.

## Reproduction and Logging

Logging is particularly useful when a bug is difficult to reproduce interactively.

Suppose a user reports:

> The cursor jumped unexpectedly after several commands.

The developer can record:

```text id="p7j4o2"
EVENT 21
    COMMAND MOVE_DOWN
    CURSOR 10 -> 11

EVENT 22
    COMMAND DELETE
    DOCUMENT lines 20 -> 19

EVENT 23
    COMMAND MOVE_DOWN
    CURSOR 11 -> 13
```

The trace provides a possible reproduction sequence:

```text id="e0j7w8"
MOVE_DOWN
DELETE
MOVE_DOWN
```

The developer can then turn that sequence into a controlled reproduction.

This creates an important relationship:

```text id="3b6j1r"
Runtime Log
     │
     ▼
Observed Failure
     │
     ▼
Extract Operation Sequence
     │
     ▼
Minimal Reproduction
     │
     ▼
Regression Test
```

## Reproduction and the Debugger

Once a bug is reproducible, the debugger becomes much more effective.

Without reproduction:

```text id="1b7v8y"
Start debugger
    ↓
Wait
    ↓
Maybe bug occurs
    ↓
Maybe not
```

With reproduction:

```text id="3n4r6a"
Known sequence
    ↓
Set breakpoint
    ↓
Run reproduction
    ↓
Stop at relevant operation
    ↓
Inspect state
```

This is why reproduction should generally precede deep debugger investigation.

The debugger answers:

> What happened during this reproduction?

It cannot compensate for an unknown or unreliable reproduction.

## From Reproduction to Regression Test

Once the cause is understood and fixed, preserve the reproduction.

For example:

```text id="9i6t0u"
Bug discovered
     ↓
Minimal reproduction
     ↓
Debug
     ↓
Fix
     ↓
Automated test
```

The regression test should capture the behavior that was previously incorrect.

For example:

```text id="h9j3sa"
test_delete_last_line_preserves_cursor_validity()
```

The name should describe the behavior being protected.

The test does not need to reproduce every detail of the original debugging session if a lower-level test can meaningfully verify the corrected behavior.

This follows the project's testing principle:

> Test behavior at the lowest level that can meaningfully verify it, and use higher-level tests for interactions lower-level tests cannot cover.

## Reproduction Should Preserve Relevant Conditions, Not Accidental Ones

A reproduction can contain accidental details.

For example:

```text id="3b4f6c"
Open file A
Resize terminal
Move cursor
Open file B
Move cursor
Delete line
```

Perhaps only:

```text id="n7g8y4"
Delete line on final line
```

is actually necessary.

The goal is not to preserve every action the original user performed.

The goal is to preserve the conditions that matter.

This distinction is important:

```text id="5r7x2a"
Original Session
    │
    ├── Relevant conditions
    └── Accidental actions
             │
             ▼
       Minimal Reproduction
```

A smaller reproduction is easier to understand, easier to test, and easier to maintain.

## When a Bug Cannot Be Reproduced

Sometimes the available evidence is insufficient.

For example:

> The editor crashed yesterday while editing a file.

This does not provide enough information to reliably reproduce the problem.

The developer should gather additional evidence:

```text id="0w3v7n"
What file?
What commands?
What terminal size?
What build?
What input?
What happened immediately before the crash?
Was there a log?
Was the crash repeatable?
```

The goal is to transform an observation into a concrete execution history.

If that cannot be done, the investigation may need to rely on logs, stack traces, core dumps, sanitizer reports, or other evidence rather than assuming a particular cause.

## Common Reproduction Mistakes

### Keeping the reproduction unnecessarily large

A long sequence makes it harder to identify the responsible operation.

### Omitting the initial state

The same commands can produce different results from different states.

### Changing the environment

A different terminal size, file, configuration, or build may hide the bug.

### Relying on timing without recording it

Timing-dependent failures require special care because simply repeating the same visible actions may not reproduce the same timing.

### Assuming one successful run proves the bug is gone

An intermittent failure may simply not have occurred during that run.

### Fixing before reproducing

Changing code before understanding the failure can make the original evidence disappear.

## Engineering Insight

A reproduction is more than a list of steps.

It is a compact description of the conditions necessary to produce an incorrect state.

Conceptually:

```text id="j5x9u4"
Initial State
      +
Environment
      +
Deterministic Operations
      │
      ▼
Incorrect State
      │
      ▼
Observable Failure
```

A good reproduction therefore has four properties:

1. **Specific** — it identifies the relevant conditions.
2. **Repeatable** — it produces the same failure reliably.
3. **Minimal** — it contains no unnecessary operations.
4. **Automatable when possible** — it can become a regression test.

This turns debugging from an anecdotal activity into an engineering process.

## Summary

Reproducing a bug means establishing the conditions and operation sequence that reliably produce the failure.

A strong reproduction identifies:

```text id="r4b2xk"
Initial State
    +
Environment
    +
Operation Sequence
    +
Expected Result
    +
Actual Result
```

The preferred process is:

```text id="6k1g3q"
Observed Bug
     ↓
Record conditions
     ↓
Reproduce
     ↓
Minimize
     ↓
Debug
     ↓
Fix
     ↓
Preserve as regression test
```

Deterministic reproduction is especially important for a stateful editor because behavior depends on accumulated state and sequences of operations. The project's testing strategy explicitly treats determinism and reproducibility as core properties of the test system.

The key principle is:

> **Before trying to explain a bug, make the bug happen reliably.**

Once the failure can be reproduced, the next challenge is determining which architectural component is responsible when the failure crosses several module boundaries. That is the subject of the next section: **Debugging Across Module Boundaries**.
