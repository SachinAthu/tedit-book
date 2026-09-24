# 38.10 Preparing the Editor for Release

A release should not begin with a last-minute build command. The editor should first be brought into a known, releasable state.

This section provides the final **release-readiness process** for the editor.

The objective is simple:

> Confirm that the software, source tree, artifacts, documentation, and release metadata are ready to be delivered.

## Release Readiness

Before creating a release, verify the major release dimensions:

```text
Release Readiness
    |
    +-- Source
    +-- Build
    +-- Tests
    +-- Installation
    +-- Documentation
    +-- Version
    +-- Artifacts
    +-- Distribution
```

These are not new subsystems.

They are the final conditions that must be satisfied before publishing the software.

## Source Readiness

The source state must be clearly identified.

Verify:

```text
[ ] Intended changes are complete
[ ] Source tree contains no unintended files
[ ] Release source state is identifiable
[ ] Required project files are present
[ ] No temporary development artifacts are included
```

The purpose is to ensure that the release is based on a deliberate source state rather than whatever happens to be present on a developer's machine.

## Build Readiness

The release should be buildable using the documented release configuration.

Verify:

```text
[ ] Release configuration is available
[ ] Clean configuration succeeds
[ ] Release build succeeds
[ ] Expected executable is produced
[ ] Target platform and architecture are correct
```

The release build should use the same controlled process established earlier in this chapter.

## Test Readiness

Before publishing, the release candidate should pass the project's required tests.

At minimum:

```text
Source
  |
  v
Build
  |
  v
Tests
  |
  +---- Failure → Stop
  |
  v
Release Candidate
```

A known failing test should not simply be ignored because the executable happens to start.

The release decision should be based on the project's defined acceptance criteria.

## Installation Readiness

The release candidate should be installed using the intended installation procedure.

Verify:

```text
[ ] Installation succeeds
[ ] Installed executable is the expected version
[ ] Required files are present
[ ] Editor starts successfully
[ ] Basic operation works after installation
```

This catches problems that may not be visible when running the editor directly from the build directory.

## Documentation Readiness

Documentation should correspond to the release being published.

Verify:

```text
[ ] Installation instructions are correct
[ ] Basic usage is documented
[ ] Commands match the release
[ ] Configuration information is current
[ ] Supported platforms are stated
[ ] Known limitations are identified
[ ] Release notes are prepared
[ ] License information is present
```

The important requirement is consistency.

```text
Release 1.0.0
    |
    +-- Software 1.0.0
    +-- Documentation 1.0.0
    +-- Release Notes 1.0.0
```

## Version Readiness

The release version should be finalized before the final artifacts are produced.

Verify:

```text
[ ] Version has been selected
[ ] Version is consistent across release metadata
[ ] Source state corresponds to the version
[ ] Executable reports the expected version
[ ] Artifact names use the expected version
```

This prevents the common failure where different parts of the release identify themselves as different versions.

## Artifact Readiness

The final release artifacts should be generated from the release build rather than assembled manually.

Verify:

```text
[ ] Correct executable is included
[ ] Required files are included
[ ] Unnecessary development files are excluded
[ ] Artifact naming is correct
[ ] Artifact can be extracted or installed
[ ] Artifact contents have been inspected
```

Where applicable, generate checksums after the final artifact has been created.

## Reproducibility Check

The release process should retain enough information to rebuild the release later.

Record the relevant release inputs:

```text
Version
Source revision
Build configuration
Compiler / toolchain
Target platform
Target architecture
Relevant dependency information
```

The purpose is not to create excessive release paperwork.

It is to preserve the information needed to understand how the release was produced.

## Final Release Candidate

At this point, the release candidate should pass through one final sequence:

```text
Release Source
      |
      v
Clean Build
      |
      v
Test
      |
      v
Package
      |
      v
Install
      |
      v
Smoke Test
      |
      v
Release Candidate
```

A **smoke test** here means a short verification that the packaged release works in the way a normal user is expected to encounter it.

For the editor, this can include:

```text
Start
  ↓
Open file
  ↓
Edit
  ↓
Save
  ↓
Exit
```

The exact smoke-test workflow should reflect the features that are actually part of the current release.

## Release Decision

After verification, the project reaches a simple decision point:

```text
              Release Candidate
                     |
              +------+------+
              |             |
          Accept          Reject
              |             |
              v             v
          Publish       Fix Problems
                            |
                            v
                     Build Again
```

A release should not be published merely because a planned date has arrived.

If a release-blocking problem is discovered, the correct action is to return to development, correct the problem, and produce a new candidate.

## Release Checklist

The complete checklist can be kept deliberately small:

```text
Source
[ ] Correct source state
[ ] No unintended files

Build
[ ] Clean release build succeeds
[ ] Correct target produced

Tests
[ ] Required tests pass
[ ] Release candidate verified

Installation
[ ] Installation succeeds
[ ] Installed editor works

Version
[ ] Version finalized
[ ] Version information consistent

Documentation
[ ] User documentation current
[ ] Release notes prepared
[ ] License included

Artifacts
[ ] Correct artifacts generated
[ ] Artifact contents verified
[ ] Checksums generated where applicable

Reproducibility
[ ] Source revision recorded
[ ] Toolchain/build information recorded

Distribution
[ ] Correct artifacts selected
[ ] Release publication prepared
```

This checklist is the practical endpoint of the chapter.

## The Release Boundary

Once these checks pass, the project has crossed an important boundary:

```text
Development
     |
     v
Release Preparation
     |
     v
Release Candidate
     |
     v
Verified Release
     |
     v
Distribution
```

After publication, the released artifact should be treated as an identified historical version rather than an editable development output.

If a defect is later discovered, the project does not silently modify the published artifact.

Instead, the defect is corrected in development and a subsequent release is produced.

```text
1.0.0
  |
  | bug discovered
  v
Fix
  |
  v
1.0.1
```

This preserves the meaning of published versions.
