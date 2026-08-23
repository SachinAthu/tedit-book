# 23.7 Recovery After a Failed Save

A safe-save strategy should not only prevent unnecessary damage. It should also define what the editor does **after a save fails**.

Consider the following situation:

```text
Document in memory
        │
        ▼
Temporary file
        │
        X
     Save fails
```

The editor now has three potentially different states:

```text
In-memory Document
        │
        ├── contains the user's latest changes
        │
        ▼
Temporary File
        │
        ├── may be incomplete
        │
        ▼
Destination File
        │
        └── should still contain the previous saved version
```

A good recovery strategy must keep these states distinct.

## The Original File Should Remain the Safe Version

The most important rule after a failed save is:

> **Do not replace a valid destination file with an incomplete or unverified temporary file.**

For example:

```text
notes.txt
    │
    ▼
Previous valid version

notes.txt.tmp
    │
    ▼
Incomplete new version
```

If writing the temporary file failed, the editor should not proceed to replacement.

Instead:

```text
Write temporary file
        │
        X
      Failure
        │
        └── Keep destination unchanged
```

The user can then decide what to do next.

## The In-Memory Document Is Still the Working Version

A failed save does not undo the user's edits. Suppose the user changed `Project deadline: Friday` to `Project deadline: Monday`, and the save fails.

The editor should continue holding the Monday version in memory while the file on disk still contains Friday:

```text
Memory
   │
   └── Latest working version — Monday

Disk
   │
   └── Last successfully saved version — Friday
```

The Dirty Flag represents this difference:

```text
Dirty = true
```

The editor must not clear it simply because a save was attempted.

## The Temporary File May Contain Useful Data

After a crash or failed save, a temporary file might contain:

```text
notes.txt.tmp
    │
    ▼
Complete or partially complete document
```

If the editor can establish that the temporary file contains a complete and valid version, it may potentially be useful for recovery.

For example:

```text
notes.txt
    │
    ▼
Old version

notes.txt.tmp
    │
    ▼
Complete new version
```

A more advanced editor could detect this situation when it starts and ask the user whether the temporary version should be recovered.

Conceptually:

```text
Editor starts
      │
      ▼
Temporary file found
      │
      ▼
Is it a valid recovery candidate?
      │
   ┌──┴──┐
   │     │
  No    Yes
   │     │
   ▼     ▼
Remove  Offer
file    recovery
```

However, this requires additional rules that our baseline editor does not currently have.

## Why Automatic Recovery Is Difficult

Finding a temporary file is easy.

Determining what it means is harder.

Suppose the editor finds:

```text
notes.txt
notes.txt.tmp
```

The temporary file could represent:

- a complete unsaved version
- an incomplete write
- an old temporary file from a previous session
- a temporary file from a save the user intentionally abandoned

The editor therefore cannot safely assume:

> "The temporary file is always the version the user wants."

A proper recovery system needs a way to distinguish these cases.

## Recovery Strategies

There are several possible approaches.

### Strategy 1 — Ignore Temporary Files

The simplest approach is to remove temporary files that are discovered after an unsuccessful save.

```text
notes.txt
notes.txt.tmp
      │
      ▼
Remove temporary file
```

Advantages:

- Simple
- Easy to reason about
- No recovery interface required

Disadvantage:

- Potentially useful unsaved data may be lost.

### Strategy 2 — Offer Recovery

A more user-friendly approach is to detect the temporary file and ask the user whether it should be recovered.

Conceptually:

```text
Temporary recovery file found.

Recover unsaved changes?

[Yes] [No]
```

Advantages:

- Gives the user control
- Can recover changes after a crash

Disadvantages:

- Requires additional startup logic
- Requires validation of the temporary file
- Requires a recovery user interface
- Introduces more states into the application

### Strategy 3 — Automatic Recovery

A more sophisticated editor could automatically restore the temporary version.

This requires a much stronger recovery protocol because the editor must be confident that the temporary file represents the latest valid state.

Automatic recovery therefore belongs to a more advanced persistence design.

## Our Baseline Decision

For the editor developed in this book, we should keep the recovery mechanism deliberately simple.

The baseline editor will:

1. Write the new document to a temporary file.
2. Replace the destination only after the temporary file has been successfully written.
3. Remove incomplete temporary files when appropriate.

The failed-save contract—keeping the Document and Dirty Flag, reporting through the Message Bar, and continuing the Event Loop—is defined in the next section.

We will **not** implement a complete automatic crash-recovery system in the baseline editor.

The important safety property is:

```text
Failed save
    │
    ▼
Original file remains protected
    │
    ▼
Document remains in memory
    │
    ▼
Dirty = true
    │
    ▼
User can try again
```

## Recovery and the Editor Lifecycle

A failed save should normally not terminate the editor and lose the latest in-memory changes. The runtime model already establishes that recoverable errors return control to the Event Loop, so the user can correct the problem—for example, by freeing disk space or choosing another file—and attempt to save again.

## Recovery Is Different from Backup

A temporary file is also not the same thing as a backup system.

A backup system might preserve:

```text
file.txt
file.txt.bak
file.txt.1
file.txt.2
```

allowing the user to recover earlier versions.

Our temporary file serves a narrower purpose:

```text
Current Document
       │
       ▼
Temporary File
       │
       ▼
Safe replacement
```

It is an intermediate artifact of the current save operation, not a historical version-management system.

## Engineering Insight

Recovery should be designed around explicit guarantees rather than assumptions.

We should be able to answer:

- What version is authoritative in memory?
- What version is authoritative on disk?
- Is the temporary file complete?
- Was the final replacement successful?
- Should the user be offered recovery?
- When can the temporary file safely be removed?

Once these questions are defined, recovery becomes an engineering problem rather than a collection of ad-hoc error handlers.

For our baseline editor, the answer is intentionally conservative:

> **Protect the last successfully saved file, preserve the user's latest changes in memory when possible, and never silently treat an incomplete temporary file as the authoritative document.**

## Summary

After a failed save, the original file should remain protected, the in-memory Document remains the authoritative working version, and the Dirty Flag stays set. A Temporary File may hold useful data, but discovering, validating, and offering it for recovery is a separate system that our baseline editor deliberately does not build.

In the next section, we will focus on the user-visible rules the editor follows when a save does not succeed: **What If the Save Fails?**
