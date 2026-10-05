# Volume IX — Advanced Topics

# Chapter 43 — Modern Text Editors: Architecture in the Real World

## Introduction

Volumes I–IX have designed a small, deliberate editor architecture from first principles. Real editors, however, look very different from one another: Nano is simple, Vim is modal, Emacs is programmable, and VS Code is a full development platform. This chapter steps outside our own design to ask:

> **Why do real editors have such different architectures, and what do those differences teach us about our own?**

The purpose is not to copy any of them. It is to study them as architectural case studies, asking how each turns input into editing operations, represents text and state, renders, and integrates extensions and language tooling. We begin by justifying the study and defining a common comparison framework. We then examine eight editors in turn, compare them directly, and distill the recurring lessons. We close by placing our own architecture in context and drawing the boundary between what we are building and what we are deliberately leaving out.

## Learning Objectives

By the end of this chapter, you should be able to:

- explain why architecture is shaped by requirements
- apply a consistent framework to compare editors architecturally rather than by feature list
- describe how Nano's simplicity, Vim's modal command language, and Neovim's API orientation differ
- explain how Emacs makes extensibility central to the editor itself
- describe how Helix combines selections, structured parsing, and language tooling
- explain how VS Code turns the editor into one subsystem of a development platform
- explain how performance becomes an architectural driver in Sublime Text and Zed
- compare the editors across interaction, document, rendering, extensibility, language, and performance
- articulate the recurring architectural lessons these systems share
- place our own architecture in context without imitating any single editor
- state what we are not building and why
