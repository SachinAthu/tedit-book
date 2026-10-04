# Volume IX — Advanced Topics

# Chapter 42 — Terminal Graphics

## Introduction

The editor built in this book is deliberately a character-based application: the renderer constructs a screen buffer of cells, and the terminal emulator interprets those cells to build the display. Modern terminals, however, can do far more—true color, mouse events, inline images, and terminal-specific graphics protocols. The central question is:

> **How can an editor distinguish its core, platform-independent behavior from the optional capabilities its terminal happens to provide?**

Terminal capabilities are not editor requirements. A terminal may support images without the editor needing to display them, and a more capable terminal does not justify a more complicated core. This chapter examines the major capabilities a modern terminal can expose, explains where each belongs in the editor's architecture, and shows why the baseline editor treats them as carefully bounded, optional extensions rather than part of its core design.

We begin by establishing the terminal model and its capability surface. We then examine true color, capability detection, images, and the Kitty graphics protocol, turn to mouse input and coordinate mapping, and consider why graphics complicate full redraw. We close by weighing the portability, security, and resource costs of advanced features, then decide whether to support terminal graphics and distill the architectural lessons.

## Learning Objectives

Readers should be able to:

- describe the traditional character-cell model and explain why a terminal capability is not an editor requirement
- explain the terminal graphics model and the difference between cell-oriented and graphics-oriented output
- describe true color as an extension of a cell's attributes rather than a new core representation
- explain why capability detection is necessary and why it belongs at the platform boundary
- explain why images introduce state and concerns beyond the ordinary screen buffer
- describe the Kitty graphics protocol as a concrete, terminal-specific extension
- explain how mouse input enters the existing input pipeline without changing the command model
- translate terminal coordinates through the window, viewport, and document coordinate spaces
- explain why graphics complicate full redraw, scrolling, and resizing
- evaluate the portability cost of advanced terminal features
- identify the security and resource concerns of terminal protocols
- evaluate when an advanced terminal capability is justified for our editor
- articulate the architectural lessons about keeping optional capabilities behind stable boundaries