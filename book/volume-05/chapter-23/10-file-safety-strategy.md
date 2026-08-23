# 23.10 Our File Safety Strategy

We have now examined the main problems involved in safely saving a document: incomplete writes, crashes, failed I/O operations, atomic replacement, durability, recovery, and the Dirty Flag.

It is time to make the design decision explicit.

Our editor will use a **full-document, temporary-file-and-atomic-replacement strategy**. This builds directly on the persistence model established by ADR-020 and the UTF-8 boundary defined by ADR-021. The Document remains the complete in-memory working state, while the File Manager is responsible for turning that state into a persistent file.

## Baseline Decision

When the user saves, our editor will:

1. Keep the complete Document in memory.
2. Encode the Document as UTF-8 at save time.
3. Write the new contents to a Temporary File.
4. Detect write and close errors before replacing the destination.
5. Replace the destination using an atomic filesystem operation.
6. Keep the Dirty Flag set if any required save step fails.
7. Report failures through the Message Bar.
8. Return to normal Event Loop operation after recoverable failures.

The resulting model is:

```text
Document in memory
        │
        │ encode
        ▼
   Temporary File
        │
        │ complete and valid
        ▼
Atomic Replacement
        │
        ├── failure ──► Dirty = true
        │
        └── success ──► Dirty = false
```

This strategy gives us a clear boundary between **editing** and **persistence**. Editing happens entirely in the in-memory Document. Saving creates a new persistent representation without immediately destroying the existing destination.

## What This Strategy Guarantees

The baseline design gives us several important guarantees:

- A partially written Temporary File does not become the destination.
- A failed save does not clear the Dirty Flag.
- The current in-memory Document remains available after a recoverable save failure.
- The user receives an error through the Message Bar.
- The Event Loop can continue after a recoverable failure.
- The destination is replaced only after the new contents have been prepared successfully.

These guarantees are more important than optimizing the save operation for every possible filesystem behavior.

## What This Strategy Does Not Guarantee

The design also has deliberate boundaries.

Atomic replacement does not by itself guarantee complete power-loss durability. Similarly, replacing a file does not necessarily preserve every piece of its original metadata.

Those concerns have been discussed separately rather than being hidden inside the basic save model.

Our goal is therefore not to claim that the editor provides every possible filesystem guarantee. Instead, we define a small set of guarantees that the implementation can clearly explain and maintain.

## Engineering Insight

A good architecture does not try to solve every possible problem at once.

Our file-safety strategy is intentionally small:

```text
Protect the old file
        +
Prepare the new file
        +
Replace atomically
        +
Only then mark the Document saved
```

This is the persistence model we will carry into the implementation volume. It is consistent with the project's existing architectural decisions rather than introducing a competing way of storing or saving Documents.

With the strategy established, we can now examine the boundaries of the design—particularly the sophisticated file-safety features that professional editors may provide but that our baseline editor intentionally does not build.

## Summary

Our editor adopts a full-document, temporary-file-and-atomic-replacement strategy: prepare the complete new version separately, replace the destination atomically, and only then clear the Dirty Flag. This yields a small, explicit set of guarantees without claiming durability properties the baseline does not provide.

In the next section, we will examine the file-safety features we deliberately do **not** build.
