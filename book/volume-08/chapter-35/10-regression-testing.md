# 35.10 Regression Testing

A text editor is not a static system.

As new features are added, existing behavior can change unintentionally. A modification to the Document may affect Cursor behavior. A change to input handling may affect commands. A change to Buffer management may affect saving or multiple Views.

A regression occurs when behavior that previously worked becomes incorrect after a change.

Regression testing provides a mechanism for detecting these failures.

The central idea is simple:

```text
Known-correct behavior
        │
        ▼
Change the system
        │
        ▼
Run existing tests
        │
        ▼
Detect unintended change
```

Regression testing therefore turns previous correctness into continuing protection.

## What Is a Regression?

A regression is not necessarily a newly introduced feature that fails.

It is often an old behavior that stops working because another part of the editor changed.

For example, suppose the editor correctly handles:

```text
Move Cursor to beginning of line
        │
        ▼
Press Backspace
        │
        ▼
Merge with previous line
```

Later, the Document implementation is modified to support another editing operation.

If Backspace at the beginning of a line now produces incorrect text, the new failure is a regression.

The original behavior was already correct. The later change unintentionally broke it.

Regression testing exists to detect precisely this kind of failure.

## Regression Tests Begin With Real Failures

A useful regression test usually originates from a concrete problem.

The development process can be represented as:

```text
Bug discovered
     │
     ▼
Understand the failure
     │
     ▼
Fix the implementation
     │
     ▼
Create a test reproducing the bug
     │
     ▼
Verify the fix
     │
     ▼
Keep the test permanently
```

The important step is creating the test.

Without it, the same defect can return later.

For example, if a failed save once caused the Dirty Flag to clear incorrectly, the corrected behavior should become a permanent test:

```text
Modify document
     │
     ▼
Save fails
     │
     ▼
Dirty Flag remains set
```

Future changes to saving, Buffer management, or state transitions can then be checked against this behavior.

## Regression Tests Should Describe the Defect

A regression test should capture the behavior that was previously broken.

Consider a defect involving line merging.

The test should not merely assert that a particular internal function was called.

Instead, it should reproduce the meaningful condition:

```text
Initial:

line one
line two

Operation:
Backspace at beginning of "line two"

Expected:

line onetwo
```

The test protects the behavior rather than the implementation.

This is important because the internal implementation may change while the expected editor behavior remains the same.

## Regression Tests Across Architectural Boundaries

Some regressions occur entirely within one component.

Others cross architectural boundaries.

For example:

```text
Input Parser
     │
     ▼
Input Engine
     │
     ▼
Command Engine
     │
     ▼
Cursor
```

Suppose an arrow-key escape sequence previously worked but stops moving the Cursor after an Input Engine change.

A regression test can preserve the complete input-to-command behavior.

Similarly:

```text
Command Engine
     │
     ▼
Buffer
     │
     ▼
File Manager
     │
     ▼
File System
```

A previous save bug can be protected with a workflow test that verifies the final file contents.

Regression tests should therefore be placed at the lowest level that reproduces the defect.

If a failure can be reproduced entirely within the Document, a Document-level test is preferable.

If it requires several components, an integration test may be appropriate.

If it requires the complete editor lifecycle, an end-to-end regression test may be necessary.

## Choosing the Correct Test Level

The relationship can be summarized as:

```text
Can the bug be reproduced in one component?
        │
       Yes
        ▼
Unit regression test

        No
        │
        ▼
Does it require multiple components?
        │
       Yes
        ▼
Integration regression test

        No
        │
        ▼
Does it require the complete application?
        │
       Yes
        ▼
End-to-end regression test
```

This prevents the regression suite from becoming unnecessarily dependent on the complete application.

A smaller test is generally easier to diagnose and faster to execute when it can adequately protect the behavior.

## Regression Tests for the Document

The Document is especially important because many editor operations depend on its behavior.

Regression tests can protect cases such as:

- inserting at the beginning of a line;
- inserting at the end of a line;
- deleting the final character;
- deleting across line boundaries;
- splitting a line;
- merging two lines;
- modifying empty lines;
- handling Unicode code points;
- preserving unrelated text.

For example, a defect in line merging might be represented as:

```text
Before:

abc
def

Action:

Backspace at column 0 of "def"

Expected:

abcdef
```

Once corrected, this behavior should remain in the regression suite.

## Regression Tests for the Cursor

Cursor behavior is another common source of regressions because changes to editing operations can affect Cursor state.

Examples include:

```text
Move right at end of line
Move left at beginning of line
Move vertically between unequal line lengths
Move vertically across empty lines
Move after line insertion
Move after line deletion
```

The preferred-column behavior is particularly important.

For example:

```text
Line 1:  abcdef
Line 2:  xy
Line 3:  abcdef
```

If vertical movement temporarily enters a shorter line, returning to a longer line should preserve the intended preferred column according to the Cursor rules established earlier.

A change to Cursor movement should therefore run these regression cases.

## Regression Tests for the Viewport

Viewport behavior can regress when rendering, Cursor movement, or terminal-size handling changes.

Important cases include:

- Cursor reaching the bottom visible row;
- Cursor moving beyond the visible region;
- scrolling back upward;
- horizontal scrolling;
- wide logical lines;
- tabs;
- empty documents;
- terminal resizing.

A regression test should focus on the visibility contract.

For example:

```text
Cursor moves below visible region
        │
        ▼
Viewport adjusts
        │
        ▼
Cursor becomes visible
```

The test need not inspect the Renderer to prove this behavior.

The Viewport itself can be tested at its own boundary, while an integration or end-to-end test can protect the complete workflow when necessary.

## Regression Tests for Input Parsing

Terminal input is particularly sensitive to changes because a small modification to the parser state machine can affect many keys.

Existing parser regressions should protect cases such as:

```text
Printable character
Ctrl-key
Escape
Arrow key
Function or special sequence
Partial escape sequence
Malformed sequence
Recovery after malformed input
```

For example, if the parser once correctly handled an arrow-key sequence split across multiple reads:

```text
Read 1: ESC [
Read 2: D
```

that behavior should remain protected.

A later change that assumes the entire sequence arrives in one read must not silently break this case.

## Regression Tests for File Safety

File operations deserve particularly strong regression protection because failures can affect user data.

Important cases include:

```text
Successful save
Failed save
Temporary-file failure
Replacement failure
Dirty Flag after successful save
Dirty Flag after failed save
```

A regression test for a successful save should verify the resulting file.

A failure test should verify that the editor does not incorrectly report success.

For example:

```text
Document modified
      │
      ▼
Save attempted
      │
      ▼
Replacement fails
      │
      ▼
Dirty Flag remains set
```

This protects the distinction between:

```text
save requested
```

and:

```text
save successfully completed
```

## Regression Tests for Multi-Buffer Editing

The multi-buffer architecture introduces additional state that must remain stable as features evolve.

A regression suite can protect behaviors such as:

```text
Edit Buffer A
     │
     ▼
Switch to Buffer B
     │
     ▼
Edit Buffer B
     │
     ▼
Return to Buffer A
     │
     ▼
Buffer A still contains its previous state
```

Similarly, if two Windows display the same Buffer, a regression test can protect the distinction between shared Buffer state and independent Window state.

For example:

```text
Shared:
    Document
    Dirty Flag
    Undo history

Per Window:
    Cursor
    Viewport
    Geometry
```

A change to one scope must not accidentally overwrite state belonging to another scope.

## Regression Tests for Configuration

Configuration can also introduce regressions.

A change to configuration loading should not unexpectedly alter established behavior.

Examples include:

- default Tab width;
- configured key bindings;
- invalid configuration handling;
- command-line overrides;
- precedence between defaults and user configuration.

For example:

```text
Built-in Defaults
       │
       ▼
User Configuration
       │
       ▼
CLI Overrides
       │
       ▼
Validated Editor State
```

A regression test can verify that a known configuration continues to produce the expected effective policy.

The purpose is not to retest every configuration rule at the application level, but to protect important behavior that has previously been established.

## Keep Regression Tests Deterministic

A regression test should fail because the editor behavior changed, not because the environment happened to behave differently.

Avoid unnecessary dependencies on:

- wall-clock timing;
- random data;
- terminal dimensions when dimensions are not relevant;
- external files that may change;
- user-specific configuration;
- unrelated environment variables.

Controlled temporary files and deterministic input sequences are preferable.

For terminal workflows, a controlled pseudo-terminal can provide a predictable environment when terminal behavior itself is part of the test.

## Regression Tests Are Not Historical Records

A regression test should remain useful after the original bug has been forgotten.

It should answer:

> What behavior must continue to work?

rather than:

> What did bug #17 do?

A useful test name might describe the behavior:

```text
failed_save_keeps_document_dirty
backspace_at_line_start_merges_lines
partial_escape_sequence_is_parsed_correctly
switching_buffers_preserves_document_state
```

These names communicate the protected contract directly.

The original issue or defect report can provide historical context, but the test itself should describe the behavior that matters.

## Avoiding a Growing Collection of Redundant Tests

Regression testing does not mean adding a new test for every possible variation of every bug.

As the editor grows, duplicate tests can make the suite difficult to understand.

Before adding a regression test, determine:

1. What behavior was broken?
2. At which architectural boundary can it be reproduced?
3. Is an existing test already protecting the same behavior?
4. What is the smallest test that would have detected the defect?

The goal is targeted protection.

For example, if a bug is already completely protected by a Document test, another end-to-end test may not add much value unless the failure involved the assembled workflow.

## Running Regression Tests After Changes

Regression tests become useful only when they are run consistently.

A practical development cycle is:

```text
Make change
    │
    ▼
Run focused tests
    │
    ▼
Run integration tests
    │
    ▼
Run regression suite
    │
    ▼
Run selected end-to-end workflows
```

The exact sequence can evolve as the project grows.

During active development, a programmer may first run tests related to the changed component.

Before considering a larger change complete, the broader suite should be executed.

This helps detect both local failures and unintended effects elsewhere in the architecture.

## Regression Testing as a Safety Net

The editor's test suite can be viewed as a growing safety net.

Initially:

```text
Small feature set
       │
       ▼
Small test suite
```

As development continues:

```text
New feature
    │
    ├──► New behavior tests
    │
    └──► Existing regression tests
```

When a defect is discovered:

```text
Defect
  │
  ▼
Fix
  │
  ▼
Regression test
  │
  ▼
Permanent protection
```

The test suite therefore grows alongside the editor.

This is especially important for an educational project because later chapters and implementation work may modify foundations established much earlier.

A change should not silently invalidate previously established behavior.

## Regression Testing and Refactoring

Refactoring changes the internal structure without intentionally changing externally observable behavior.

Regression tests provide evidence that this property is being maintained.

For example, the internal representation of the Document could eventually change while preserving the same logical editing behavior.

Before refactoring:

```text
Document implementation A
        │
        ▼
Expected behavior
```

After refactoring:

```text
Document implementation B
        │
        ▼
Same expected behavior
```

If the relevant regression tests continue to pass, they provide evidence that the refactoring preserved the tested contract.

This is one of the most important practical benefits of testing: tests allow implementation details to evolve without requiring every architectural decision to remain permanently frozen.

## The Role of Regression Testing in This Editor

Regression testing connects the past and present states of the project.

Unit tests protect component behavior.

Integration tests protect architectural boundaries.

End-to-end tests protect complete workflows.

Regression tests preserve important behaviors from all of these levels after the behavior has been established or after a defect has been corrected.

The resulting relationship is:

```text
                 ┌─────────────────┐
                 │ Existing Tests  │
                 └────────┬────────┘
                          │
                          ▼
                     Code Change
                          │
                          ▼
                  Run Test Suite
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
          Still Works             Regression
              │                       │
              ▼                       ▼
        Preserve behavior       Investigate
                                      │
                                      ▼
                                   Fix + Test
```

A mature editor is not merely a collection of features. It is a collection of features whose previously established behavior is continually protected.

## Key Principle

> Regression testing preserves previously established editor behavior: when a defect is discovered or the implementation changes, the affected behavior should become protected by an appropriate unit, integration, or end-to-end test so that future changes do not silently reintroduce the problem.

## Summary

Regression testing turns previously established correctness into permanent protection. A useful regression test originates from a real failure, is placed at the lowest level that reproduces the defect, describes the protected behavior rather than the implementation, and remains deterministic and targeted to avoid an unwieldy suite.

The next section examines the failure conditions that these tests must cover: **Section 35.11, Testing Failure Cases**.
