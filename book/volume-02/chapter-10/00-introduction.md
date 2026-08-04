# Volume II — Software Architecture

# Chapter 10 — Project Structure

## Introduction

In the previous chapters, we designed the editor from an architectural perspective. We identified the major components of the system, defined their responsibilities, established clear dependency rules, and examined how the application executes through its event-driven runtime model. At this point, we understand **what** the editor consists of and **how** its components interact during execution.

The next step is determining **how that architecture should be organized as a software project**.

Software architecture and project structure are closely related, but they are not the same. Architecture describes the logical organization of a system—its components, responsibilities, and interactions. Project structure describes how that architecture is represented in source code through directories, modules, header files, source files, and build configuration. A well-designed project structure makes the architecture visible to anyone reading the code.

As software projects grow, organization becomes increasingly important. Poorly organized code makes it difficult to locate functionality, understand dependencies, and extend the system without introducing defects. A clear project structure, on the other hand, improves readability, maintainability, and collaboration by ensuring that each module has a well-defined purpose and communicates with other modules through carefully designed interfaces.

In this chapter, we will design the physical layout of our editor project. We will examine how architectural components become source code modules, how responsibilities are divided between header and source files, how modules expose public interfaces while hiding implementation details, and how dependency rules are enforced through the project structure. These organizational decisions will provide the foundation for every implementation chapter that follows.

Although no editor functionality will be implemented in this chapter, the decisions we make here will shape the maintainability, extensibility, and clarity of the entire codebase.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain the difference between software architecture and project structure.
- Understand why a well-organized project is easier to develop, maintain, and extend.
- Organize a C project into logical directories and modules based on architectural responsibilities.
- Distinguish between the responsibilities of header files and source files.
- Design clean module interfaces that promote low coupling and information hiding.
- Apply dependency rules to prevent unnecessary coupling and circular dependencies.
- Understand the role of a build system in transforming source code into an executable program.
- Follow consistent coding and naming conventions that improve readability and maintainability.
