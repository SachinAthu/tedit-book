# Volume II — Software Architecture

# Chapter 9 — Event Driven Architecture

## Introduction

In the previous chapter, we designed the architecture of the editor by identifying its major components, defining their responsibilities, and establishing how they depend on one another. While that architecture describes the structure of the system, it does not explain how the editor actually operates once it starts running.

A text editor is an **event-driven application**. Rather than executing a fixed sequence of operations and terminating, it continuously waits for events, responds to user input, updates its internal state, and refreshes the display. This cycle repeats throughout the lifetime of the application, allowing the editor to remain interactive and responsive.

In this chapter, we examine the runtime behavior of the editor. We will explore the event loop, follow the flow of an event through the system, and see how the editor's components work together to process input, update the editor state, and render each frame. Understanding this execution model provides the foundation for the implementation chapters that follow.

## Learning Objectives

By the end of this chapter, you will be able to:

- Explain what an event-driven application is and why text editors use this architecture.
- Describe the purpose and structure of the event loop.
- Identify the different types of events processed by the editor.
- Trace the flow of an event from keyboard input to screen rendering.
- Explain how the editor state is updated in response to events.
- Understand the relationship between event processing, state management, and rendering.
- Recognize why this execution model results in a responsive and maintainable editor design.
