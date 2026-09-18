# Volume VIII — Software Engineering

# Chapter 35 — Testing

## Introduction

Throughout the book, we have built the editor from components with precise responsibilities: the Document owns text, the Cursor owns position, the Viewport owns visibility, the Input Parser owns terminal bytes, and the Command Engine coordinates them all. Up to now, correctness has been argued mostly through design. This chapter changes that: it makes correctness **verifiable**.

Testing is particularly important for a stateful, interactive system. A component can compile, appear to work, and still corrupt text, move the Cursor beyond valid bounds, or leave the Dirty Flag wrong after a failed save. Testing converts requirements into observable checks and protects the architecture as the editor grows.

This chapter builds the testing model progressively:

```text
Motivation — why testing matters for a stateful editor
    ↓
Strategy — unit, integration, and end-to-end testing
    ↓
Unit Tests — the Editor State components, one by one
    ↓
Integration — contracts and boundaries between components
    ↓
End-to-End — complete user-visible workflows
    ↓
Protection — regressions, failure cases, and organization
    ↓
Reflection — invariants and the mistakes to avoid
```

We begin by explaining why testing matters and defining an overall strategy. We then apply unit testing to the Document, Cursor, Viewport, and Input Parser, move to integration and end-to-end testing, and extend the model with regression, failure, and organizational concerns. We close by testing the invariants that hold the architecture together and by reviewing the common mistakes that undermine a test suite.

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain why compilation and manual experimentation are not enough to establish correctness.
- Describe a three-level testing strategy and what each level does and does not prove.
- Explain what makes a unit test focused, deterministic, and independent of the rest of the editor.
- Design tests for the Document, Cursor, Viewport, and Input Parser at their own boundaries.
- Explain how integration tests verify contracts between components.
- Describe how end-to-end tests verify complete user-visible workflows.
- Explain how regression tests preserve previously established behavior.
- Describe how failure-case testing protects valid state and user data.
- Organize a test suite around responsibilities, boundaries, and workflows.
- Recognize common testing mistakes and how to avoid them.
- Explain how testing invariants protects the architecture as the editor changes.