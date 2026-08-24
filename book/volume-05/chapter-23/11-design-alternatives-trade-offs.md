# 23.11 Design Alternatives and Trade-offs

There is more than one way to save a document safely. The strategies differ in how much protection they provide, but also in how much state and implementation complexity they introduce.

| Strategy                | Simplicity |                  Safety | Complexity |
| ----------------------- | ---------: | ----------------------: | ---------: |
| Direct overwrite        |       High |                     Low |        Low |
| Temporary + replacement |     Medium |                    High |     Medium |
| Backup + replacement    |     Medium | Higher recovery options |     Medium |
| Journaling              |        Low |               Very high |       High |
| Versioned persistence   |        Low |               Very high |       High |

## Direct Overwrite

The simplest approach is to open the destination file and write the new contents directly.

```text
Document
   │
   ▼
Destination File
```

Its main advantage is simplicity. There is no Temporary File and no additional replacement step.

The problem is that a failure during the write can leave the destination partially updated. The previous valid version may therefore be lost.

For our editor, the safety trade-off is not acceptable.

## Temporary File + Atomic Replacement

Our chosen strategy prepares the new version separately:

```text
Document
   │
   ▼
Temporary File
   │
   ▼
Atomic Replacement
   │
   ▼
Destination
```

This adds a few steps, but provides a substantial improvement in safety. The original destination is not replaced until the new contents have been successfully prepared.

It also fits naturally with our existing full-document persistence model and keeps the recovery system relatively small.

## Backup + Replacement

Another approach is to create a backup of the existing file before replacing it:

```text
Original
   │
   ├──► Backup
   │
   └──► New Version
```

This provides an additional recovery option. If the new version is later found to be incorrect or a failure occurs, the backup may provide an older copy.

However, this introduces another persistent artifact and raises additional questions about backup naming, cleanup, retention, and recovery.

Our baseline editor does not require this functionality.

## Journaling

A journaling system records changes or save information separately from the main file.

Conceptually:

```text
Document
   │
   ├──► Journal
   │
   └──► Persistent File
```

This can provide very strong recovery capabilities, particularly after crashes. However, the editor must maintain, validate, and replay journal information correctly.

That is significantly more complex than the persistence model required by our project.

## Versioned Persistence

A versioned system maintains multiple persistent versions of a document.

```text
Version 1
Version 2
Version 3
Version 4
   │
   ▼
Current Version
```

This provides excellent recovery and history capabilities, but requires policies for version creation, storage, retention, cleanup, and restoration.

It is therefore well beyond the requirements of our baseline single-document editor.

## Why We Choose Temporary File + Atomic Replacement

Our goal is not to implement the most sophisticated persistence system possible.

We want a design that provides a **substantial safety improvement over direct overwrite while remaining simple enough to understand and implement from first principles**.

Temporary-file creation followed by atomic replacement provides that balance:

```text
                 File Safety
                     │
        ┌────────────┴────────────┐
        │                         │
   Direct overwrite       Temporary + replacement
        │                         │
     simpler                    safer
        │                         │
        └────────────┬────────────┘
                     │
              Our baseline
```

It protects the previous destination while the new version is being prepared, preserves the in-memory Document when saving fails, and does not require a complete recovery subsystem.

The trade-off is deliberate:

> **We accept moderate implementation complexity to obtain a significant improvement in file safety.**

More sophisticated approaches remain possible future extensions, but they are not justified by the baseline requirements of this editor.

## Summary

There are several ways to save files safely—direct overwrite, temporary file plus atomic replacement, backup, journaling, and versioned persistence. Our editor chooses temporary file plus atomic replacement because it provides a substantial safety improvement over direct overwrite while remaining simple enough to implement from first principles.

This completes the file-safety design. The **chapter summary** consolidates the key concepts, common mistakes, and exercises for this chapter.
