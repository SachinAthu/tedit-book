# 35.2 Testing Strategy

Testing the editor requires more than writing a collection of individual tests. The editor contains several components with different responsibilities, and each component can fail in different ways.

A useful testing strategy therefore begins by deciding **what should be tested, at which level, and for what purpose**.

Our architecture already provides a natural structure for this.

```text
                         Editor
                           │
             +-------------+-------------+
             │             │             │
             ▼             ▼             ▼
           Unit       Integration     End-to-End
          Testing       Testing         Testing
             │             │             │
             ▼             ▼             ▼
        Components     Components       Complete
        in isolation   working together  workflows
```

The goal is not to choose one testing technique. The goal is to use different levels of testing to answer different questions.

## Testing at Different Levels

For our editor, the main levels are:

```text
Unit Tests
    │
    ├── Document
    ├── Cursor
    ├── Viewport
    └── Input Parser
          │
          ▼
Integration Tests
    │
    ├── Input → Command
    ├── Command → State
    ├── File Manager → Document
    └── Renderer → Screen Buffer
          │
          ▼
End-to-End Tests
    │
    └── User interaction → complete result
```

Each level provides a different kind of evidence.

### Unit testing asks:

> Does this component behave correctly by itself?

### Integration testing asks:

> Do these components cooperate correctly?

### End-to-end testing asks:

> Does the editor behave correctly when used as a complete system?

These questions overlap, but they are not interchangeable.

## Unit Tests: Verify Local Behavior

Unit tests should form the foundation of the testing strategy.

A unit test should isolate a small piece of behavior and provide controlled input.

For example:

```text
Document
    │
    │ insert("X", row=0, column=3)
    ▼
Expected Document
```

The test does not need a terminal.

It does not need keyboard input.

It does not need the Renderer.

It only needs to verify the behavior of the Document operation.

This makes the test:

- fast
- deterministic
- easy to diagnose
- independent of unrelated components

The same principle applies to Cursor and Viewport behavior.

## Integration Tests: Verify Boundaries

Individual components can each be correct while their interaction is incorrect.

Consider:

```text
Input Parser
     │
     ▼
Input Engine
     │
     ▼
Command Engine
```

Suppose the Input Parser correctly produces:

```text
KEY_ARROW_LEFT
```

and the Command Engine correctly handles:

```text
MOVE_CURSOR_LEFT
```

The system can still fail if the Input Engine maps the key event to the wrong command.

Unit tests for the individual components might all pass.

An integration test can expose the problem:

```text
KEY_ARROW_LEFT
       │
       ▼
Input Engine
       │
       ▼
MOVE_CURSOR_LEFT
       │
       ▼
Cursor moves left
```

Integration testing therefore concentrates on **contracts between components**.

This is particularly important in our architecture because responsibilities are deliberately separated across modules. The architecture defines the Input Handler, Input Parser, Input Engine, Command Engine, Editor State, Renderer, and File Manager as distinct responsibilities.

## End-to-End Tests: Verify Complete Behavior

At the highest level, we want to know whether the editor works as a complete system.

Consider saving a modified document:

```text
User
 │
 ▼
Ctrl-S
 │
 ▼
Input Handler
 │
 ▼
Input Parser
 │
 ▼
Input Engine
 │
 ▼
Command Engine
 │
 ▼
File Manager
 │
 ▼
File
```

An end-to-end test can verify the complete outcome:

```text
Initial file:
Hello

User edits:
Hello World

User saves

Expected file:
Hello World
```

This test does not care which internal function performed each operation. It verifies the behavior from the perspective of the complete system.

That makes end-to-end tests valuable, but also more expensive and harder to diagnose when they fail.

## Testing the Right Things at the Right Level

A common mistake is to test everything at the highest level.

For example, suppose `document_insert()` is incorrect.

We could discover the problem through a full terminal test:

```text
Start editor
    ↓
Open file
    ↓
Move cursor
    ↓
Type character
    ↓
Save
    ↓
Read file
    ↓
Failure
```

The test tells us that something is wrong.

But it does not immediately tell us whether the problem is in:

```text
Input Parser
Input Engine
Command Engine
Document
File Manager
```

A unit test for `document_insert()` would isolate the problem much more quickly.

Therefore:

> Test behavior at the lowest level that can meaningfully verify it, and use higher-level tests to verify the interactions that lower-level tests cannot cover.

## Deterministic Tests

Editor tests should preferably be deterministic.

Given the same initial state and the same operation sequence:

```text
Initial State
     +
Operations
     ↓
Expected Result
```

the test should produce the same result every time.

For example:

```text
Initial Document:

Hello
World

Operation:

document_insert('X', 0, 5)

Expected:

HelloX
World
```

There should be no dependency on:

- terminal size
- terminal appearance
- execution timing
- user's keyboard
- current working directory
- random data

unless those things are specifically what the test is intended to verify.

Determinism makes failures reproducible.

## Test the Boundaries

Editor bugs frequently occur at boundaries rather than in ordinary cases.

For a Document:

```text
Beginning
    │
    ▼
Middle
    │
    ▼
End
```

For a Cursor:

```text
Row 0
Column 0
Last row
Last column
```

For a Viewport:

```text
Top edge
    │
Visible region
    │
Bottom edge
```

For input:

```text
Single byte
    │
Multi-byte character
    │
Escape sequence
    │
Incomplete sequence
```

Testing should therefore deliberately include boundary conditions.

For example, testing:

```text
document_insert()
```

only in the middle of a normal line gives limited confidence.

A stronger test set includes:

```text
Insert at beginning
Insert in middle
Insert at end
Insert into empty line
Insert Unicode character
```

The exact cases will be developed in the later sections of this chapter.

## Test State Transitions, Not Just Final State

An editor is stateful.

The final state alone may not reveal whether the correct transitions occurred.

Consider:

```text
Document
   │
   ▼
Edit
   │
   ▼
Dirty = true
   │
   ▼
Save
   │
   ▼
Dirty = false
```

A test should verify not only that the file contains the correct contents, but also that the Dirty Flag changes at the correct point.

The save process defined by the project's file-safety strategy is especially important here:

```text
Prepare temporary file
        │
        ▼
Write document
        │
        ▼
Atomic replacement
        │
        ▼
Success
        │
        ▼
Dirty = false
```

If replacement fails:

```text
Atomic replacement
        │
        ▼
Failure
        │
        ▼
Dirty = true
```

A test that checks only the resulting file could miss an incorrect Dirty Flag.

Testing should therefore verify important **state transitions** as well as final outputs.

## Test Architectural Invariants

Some tests should protect architectural rules, not merely individual features.

For example:

### Renderer must not modify the Document

A rendering test can capture the Document state before rendering:

```text
Before Render
      │
      ▼
   Renderer
      │
      ▼
After Render
```

The Document should be unchanged.

### Buffer switching must preserve Buffer state

With multiple Buffers:

```text
Buffer A
Cursor = (4, 8)

        ↓ switch

Buffer B
Cursor = (1, 2)

        ↓ switch back

Buffer A
Cursor = (4, 8)
```

The test verifies that Cursor state belongs to the appropriate Window/View rather than accidentally becoming global state.

### Failed saves must preserve the Dirty Flag

```text
Dirty = true
     │
     ▼
Save fails
     │
     ▼
Dirty = true
```

These tests protect architectural decisions that would otherwise be easy to break during later development.

## Manual Testing Still Has a Place

Automated tests cannot replace every form of testing.

A terminal editor has behavior that is difficult to completely represent through isolated automated tests.

For example:

- Does the terminal display look correct?
- Is the cursor visually positioned correctly?
- Does the status bar remain readable?
- Does the editor behave correctly after resizing the terminal?
- Does the terminal return to normal after exit?

Manual testing is useful for these cases.

The distinction is:

```text
Automated Tests
    │
    ├── Repeatable
    ├── Fast
    └── Precise

Manual Tests
    │
    ├── Visual
    ├── Interactive
    └── Exploratory
```

Manual testing should therefore complement the automated suite rather than replace it.

## The Testing Strategy for Our Editor

Putting everything together:

```text
                         Testing Strategy
                                │
          +---------------------+---------------------+
          │                     │                     │
          ▼                     ▼                     ▼
      Unit Tests          Integration Tests     End-to-End Tests
          │                     │                     │
          ▼                     ▼                     ▼
   Local correctness       Component contracts    User workflows
          │                     │                     │
          +---------------------+---------------------+
                                │
                                ▼
                         Manual Testing
                         Visual / Interactive
```

And across all levels:

```text
                  ┌──────────────────────┐
                  │  Requirements        │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │ Expected Behavior    │
                  └──────────┬───────────┘
                             │
              +--------------+--------------+
              │              │              │
              ▼              ▼              ▼
             Unit       Integration    End-to-End
              │              │              │
              +--------------+--------------+
                             │
                             ▼
                    Confidence in System
```

The strategy is therefore not simply **"write lots of tests."**

It is:

1. Identify important behavior.
2. Test local behavior at the component level.
3. Test contracts between components.
4. Test complete user workflows.
5. Test failure and boundary conditions.
6. Preserve discovered bugs as regression tests.
7. Use manual testing where automated tests cannot adequately represent the terminal experience.

The next sections apply this strategy to the editor's individual components, beginning with **Unit Testing** and then examining the Document, Cursor, Viewport, and Input Parser in detail.

## Summary

The testing strategy combines unit, integration, and end-to-end tests, supported by manual testing where automation cannot represent the terminal experience. Tests should be deterministic, deliberately cover boundaries, transitions, and failure conditions, protect architectural invariants, and place each behavior at the lowest level that can meaningfully verify it.

The next section describes the foundation of this strategy: **Section 35.3, Unit Testing**.
