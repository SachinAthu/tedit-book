### 38.1 Development Builds

Focus on the build used **while developing the editor**.

Cover:

- What a development build is
- Debug information
- Assertions
- Diagnostic logging
- Sanitizers
- Compiler warnings
- Optimization levels
- Development build goals
- Why development and release builds should differ

**Do not repeat Chapter 36's debugging techniques.**
Chapter 36 explains _why and how debugging works_. This section only explains how those facilities are selected and assembled into a development build.

This distinction is already supported by ADR-029, which defines the debug build as including assertions, symbols, reduced optimization, diagnostics, and sanitizers.

---

### 38.2 Release Builds

Focus on the characteristics of a **production-ready executable**.

Cover:

- What distinguishes a release build
- Optimization
- Disabled development diagnostics
- Compiler warnings remaining enabled
- Stripping unnecessary debugging information
- Build-time vs runtime diagnostics
- Release correctness requirements
- Why "release" does not mean "remove safety"

Important distinction:

```text
Development Build
    ↓
Maximum observability

Release Build
    ↓
Maximum usability / appropriate optimization
```

Do **not** discuss debugging itself here. That belongs to Chapter 36.

---

### 38.3 CMake Build Configurations

This is where CMake returns, but **as a build/distribution mechanism rather than as an architectural decision**.

Chapter 10 already established the decision to use CMake. ADR-012 explicitly records that decision and why it was made.

Therefore, do **not** repeat:

- Why CMake was selected
- CMake vs Make
- General CMake introduction
- Project structure

Instead cover:

- CMake configuration types
- Debug/Release builds
- Build directories
- Out-of-source builds
- Compiler configuration
- Platform-specific configuration
- Build targets
- Installation targets
- Testing integration
- Packaging integration
- A clean build workflow

The conceptual boundary is:

```text
Chapter 10
    "Why did we choose CMake?"

Chapter 38
    "How do we use CMake to produce software?"
```

This is an important distinction.

---

### 38.4 Installation

This deserves its own section because **building software and installing software are different operations**.

Cover:

- What installation means
- Installation prefix
- Executable location
- Configuration files
- Documentation
- Platform conventions
- User installation vs system installation
- CMake installation rules
- Installation layout

Conceptually:

```text
Source Tree
     │
     ▼
Build
     │
     ▼
Executable
     │
     ▼
Install
     │
     ├── bin/
     ├── share/
     └── documentation
```

Do not turn this into a Linux filesystem tutorial. The purpose is to teach the editor's installation model.

---

### 38.5 Release Artifacts

This should focus on **what is actually delivered to users**.

Cover:

- Executable
- Supporting files
- Documentation
- License
- Configuration examples
- Source archive
- Binary archive
- Checksums
- Debug symbols as optional developer artifacts

Distinguish:

```text
Build Output
    ≠
Release Artifact
```

A compiler producing `editor` does not automatically mean you have a complete release.

This section is particularly useful because it creates the bridge between the technical build and actual software distribution.

---

### 38.6 Versioning

Focus on identifying releases and communicating compatibility.

Cover:

- Why software needs versions
- Version numbers
- Semantic versioning as a model
- Major/minor/patch meaning
- Development versions
- Release tags
- Version embedded in the executable
- Version displayed by `--version`
- Relationship between source revision and release

Avoid turning this into a generic Git chapter.

The important question is:

> How does a user identify exactly which version of the editor they have?

---

### 38.7 Documentation

This section should cover **documentation required for a releasable editor**, not general technical writing.

Cover:

- README
- Installation instructions
- Usage instructions
- Configuration documentation
- Keyboard shortcuts
- Supported platforms
- Known limitations
- Troubleshooting
- Changelog
- License
- Release notes

A useful distinction:

```text
Developer Documentation
        │
        └── How the software is built

User Documentation
        │
        └── How the software is used

Release Documentation
        │
        └── What changed in this version
```

This avoids overlapping with the book itself.

---

### 38.8 Distribution

Now move from **creating artifacts** to **getting them to users**.

Cover:

- Source distribution
- Binary distribution
- Archive-based distribution
- Package-manager distribution
- Platform-specific distribution
- Distribution channels
- Checksums and integrity
- Release announcements
- Choosing a distribution strategy

Keep this conceptual.

Do **not** go deeply into specific package managers or repository infrastructure. Those would distract from the editor architecture and can become outdated quickly.

---

### 38.9 Reproducible Builds

I strongly recommend keeping this section.

It is more advanced than the previous sections, but it fits the **software-engineering purpose of Volume VIII**.

Cover:

- What reproducible builds mean
- Why two builds should ideally produce equivalent artifacts
- Sources of build variation
- Timestamps
- Compiler versions
- Build paths
- Environment variables
- Dependency versions
- Deterministic archives
- Recording the build environment
- Reproducibility vs identical byte-for-byte output

Important distinction:

```text
Repeatable build
    ↓
Same procedure can be executed again

Reproducible build
    ↓
Independent environments can produce
equivalent / identical artifacts
```

This should remain conceptual rather than becoming a tool-specific reproducible-build tutorial.

---

### 38.10 Preparing the Editor for Release

This should **not simply repeat 38.1–38.9**.

Instead, make it a release-readiness process that brings everything together.

Cover:

1. Source is clean
2. Tests pass
3. Regression tests pass
4. Debugging instrumentation is not accidentally required
5. Release configuration builds successfully
6. Supported platforms are verified
7. Version is assigned
8. Documentation is complete
9. Installation is verified
10. Release artifacts are generated
11. Artifacts are checked
12. Release is reproducible/documented
13. Final release is published

A useful final pipeline:

```text
Source
  │
  ▼
Tests
  │
  ▼
Release Build
  │
  ▼
Installation Test
  │
  ▼
Release Artifacts
  │
  ▼
Verification
  │
  ▼
Distribution
```

This gives Chapter 38 a strong conclusion without introducing another technical subsystem.

---
