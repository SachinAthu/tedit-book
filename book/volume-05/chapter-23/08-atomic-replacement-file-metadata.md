# 23.8 Atomic Replacement and File Metadata

Atomic replacement solves one important problem:

> **How do we replace the file contents without unnecessarily exposing a partially written destination file?**

It does not automatically solve every problem associated with replacing a file.

When the original file is replaced by a Temporary File, filesystem metadata may also be affected. Depending on the operating system, filesystem, and replacement strategy, properties such as these may change:

- Permissions
- Ownership
- Timestamps
- Other file attributes

For example, the original file might have specific permissions:

```text
document.txt
Permissions: 0644
```

A newly created Temporary File may have different metadata. After replacement, the resulting `document.txt` may therefore not have exactly the same metadata as the original.

## Why Replacing a File Changes Its Metadata

The reason lies in how files are stored. A directory entry—the name `document.txt`—points to an **inode**, the filesystem object that holds the file's contents together with its metadata: permissions, ownership, timestamps, and other attributes.

A direct overwrite writes through the same inode, so the file's permissions and ownership survive the save:

```text
Document
    │
    ▼ write in place
document.txt ──► inode A ──► same metadata (permissions, ownership)
```

An atomic replacement works differently. The editor never edits the destination's inode. Instead, `rename()` changes the directory entry to point at the **Temporary File's inode**, which carries metadata of its own:

```text
Before:
document.txt     ──► inode A ──► old contents, original metadata
document.txt.tmp ──► inode B ──► new contents, Temporary File's metadata

After rename():
document.txt     ──► inode B ──► new contents, Temporary File's metadata
```

The original inode is no longer referenced by the directory entry, so the original file's metadata does not survive the replacement. The resulting `document.txt` therefore reflects the Temporary File's permissions, ownership, and timestamps rather than the previous version's.

This leads to an important distinction:

```text
Safely replacing file contents
            ≠
Preserving every property of the original file
```

## Engineering Insight

For our editor, the primary file-safety requirement is **protecting the document contents during saving**.

We therefore do not need to build a sophisticated metadata-preservation system into the baseline implementation. The safe-save design will focus on correctly writing the new Document and atomically replacing the destination.

Metadata preservation can be treated as a separate concern and revisited later if the editor's requirements demand it.

This is an important engineering principle:

> **A reliable design explicitly defines which guarantees it provides—and which guarantees it intentionally leaves for later.**

For the baseline editor, we care primarily about:

```text
Complete Document
        +
Safe replacement
        +
Correct Dirty Flag
        +
Error reporting
```

More advanced behavior, such as carefully preserving ownership, permissions, timestamps, or platform-specific file attributes, is deferred.

This keeps the implementation aligned with the project's broader goal: build a safe and understandable persistence mechanism first, then introduce additional complexity only when a concrete requirement justifies it.

## Summary

Replacing a file atomically protects the contents but does not necessarily preserve the original file's metadata, such as permissions, ownership, and timestamps. Our baseline editor therefore focuses its guarantees on the complete Document, safe replacement, the Dirty Flag, and error reporting, leaving metadata preservation as a deferred concern.

In the next section, we will consolidate these ideas into the editor's explicit **file safety strategy**.
