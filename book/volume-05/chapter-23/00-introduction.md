# Volume V — File System

# Chapter 23 — File Safety

## Introduction

In Chapter 21, we introduced the **File Manager** and the basic lifecycle of document persistence: the editor loads a file into the in-memory Document, and later writes the complete Document back to disk when the user saves.

But saving is more than a sequence of successful system calls. A save operation can be interrupted at any moment, by an application crash, a power failure, a full disk, a permissions problem, or an I/O error. The simplest approach—opening the destination and rewriting it—can destroy the previous valid version before the new one is known to be complete.

This leads to the central question of this chapter:

> **What state is the file left in when the save fails?**

A professional design should not simply ask whether the save operation completed. It must also consider what the file contains if the operation fails, and it should protect the last known-good version for as long as possible.

In this chapter, we will build a safer save strategy step by step, so that a failed save does not unnecessarily destroy the user's existing data and the editor fails in a controlled way. The goal is a practical, understandable strategy—not a complete crash-recovery system or a database-grade persistence engine.

## Learning Objectives

Readers should be able to:

- explain why directly overwriting the destination during a save is dangerous
- identify the different ways a save operation can fail
- explain how a temporary file separates constructing the new version from replacing the old one
- describe what atomic replacement means and why `rename()` provides it
- distinguish **atomicity** from **durability**
- describe what happens to the original file, Temporary File, and in-memory Document after crashes at different stages of a save
- explain the editor's recovery policy and why the baseline editor stops short of full automatic recovery
- state what the editor must do when a save fails: keep the Document, keep the Dirty Flag set, report the error, and continue the Event Loop
- explain why atomic replacement does not by itself preserve every property of the original file
- summarize the guarantees of our baseline file-safety strategy
- describe the file-safety features our baseline editor intentionally does not build
- compare alternative safe-save designs and justify the editor's chosen trade-off