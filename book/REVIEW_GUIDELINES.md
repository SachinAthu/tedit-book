# REVIEW_GUIDELINES.md

**Project:** Building a Terminal Text Editor  
**Version:** 1.0  
**Status:** Active

---

# Purpose

This document defines the review process for every chapter in the book.

Its purpose is to ensure the manuscript remains:

- technically accurate
- internally consistent
- educationally effective
- stylistically uniform
- publication ready

A chapter is considered **complete** only after it passes every review stage described in this document.

---

# Review Philosophy

Writing and reviewing are separate activities.

Writing focuses on teaching.

Reviewing focuses on improving.

Never attempt to write and review simultaneously.

Every review should have a single objective.

---

# Review Pipeline

Every chapter must pass through the following stages.

Draft

↓

Technical Review

↓

Revision

↓

Copy Edit

↓

Revision

↓

Reader Simulation

↓

Revision

↓

Final Acceptance

↓

Git Commit

↓

Freeze

Once frozen, a chapter should only be modified to correct factual errors or maintain consistency with later editorial decisions.

---

# Stage 1 — Draft

Objective:

Produce the best possible first draft.

Requirements:

- complete chapter
- follows Book-Bible.md
- follows outline.md
- follows ADRs.md
- complete explanations
- examples included
- diagrams included where appropriate
- summary included
- next chapter preview included

Do not worry about perfection.

The goal is completeness.

---

# Stage 2 — Technical Review

Purpose:

Verify technical correctness.

Review against:

- Book-Bible.md
- outline.md
- ADRs.md
- previous chapters

Check for:

## Technical Accuracy

- incorrect facts
- misleading explanations
- missing assumptions
- incorrect terminology
- unsupported claims

---

## Architecture Consistency

Ensure the chapter agrees with all Architecture Decision Records.

No contradictions.

---

## Terminology

Verify consistent use of:

- Editor State
- Text Buffer
- Renderer
- Cursor
- Viewport
- Frame
- Event Loop
- Raw Mode
- etc.

Never introduce alternate names.

---

## Educational Order

Concepts should progress from:

Problem

↓

Motivation

↓

Theory

↓

Example

↓

Implementation discussion

↓

Trade-offs

---

## Scope

Check that the chapter does not:

- teach future concepts prematurely
- repeat previous chapters unnecessarily
- omit essential prerequisites

---

Deliverables

The reviewer should report:

Critical Issues

Recommended Improvements

Minor Suggestions

No rewriting.

Only review.

---

# Stage 3 — Revision

Apply only accepted review comments.

Every change should preserve:

- chapter flow
- terminology
- writing style

Avoid unnecessary rewriting.

---

# Stage 4 — Copy Edit

Purpose:

Improve language without changing meaning.

Check:

- grammar
- punctuation
- spelling
- sentence flow
- formatting
- heading hierarchy
- markdown consistency
- capitalization
- list formatting

No technical changes.

---

# Stage 5 — Reader Simulation

Purpose:

Simulate an intelligent first-time reader.

The reviewer should identify:

- confusing explanations
- unexplained terminology
- missing examples
- missing diagrams
- logical jumps
- unclear transitions
- unanswered "why?" questions

Focus on comprehension.

Not grammar.

---

# Stage 6 — Final Acceptance

Before approval, verify:

✓ Chapter title matches outline

✓ Section order is correct

✓ Terminology is consistent

✓ Book Bible followed

✓ ADRs followed

✓ No technical inaccuracies

✓ Examples are accurate

✓ Diagrams are correct

✓ Cross references are correct

✓ Summary included

✓ Next chapter preview included

---

# Acceptance Checklist

A chapter cannot be merged unless all answers are YES.

## Structure

□ Introduction exists

□ Learning Objectives exist

□ Main content complete

□ Summary included

□ Next chapter preview included

---

## Technical

□ No factual errors

□ No contradictions

□ Correct terminology

□ Correct complexity discussions

□ Correct architecture discussions

---

## Educational

□ Concepts introduced gradually

□ Examples support explanations

□ Diagrams improve understanding

□ Reader can follow progression

□ No missing prerequisites

---

## Style

□ Matches Book Bible

□ Professional tone

□ Consistent formatting

□ Paragraphs readable

□ Headings consistent

---

## Repository

□ Cross references correct

□ ADR references correct

□ No duplicated explanations

□ No repeated diagrams

□ No conflicts with previous chapters

---

# Review Severity Levels

## Critical

Must be fixed before merge.

Examples:

- incorrect technical information
- contradiction
- broken explanation
- architectural inconsistency

---

## Major

Should be fixed.

Examples:

- weak explanation
- poor organization
- confusing examples
- missing concept

---

## Minor

Optional improvements.

Examples:

- wording
- formatting
- readability
- smoother transitions

---

# Quality Scoring

Reviewers should assign scores.

Technical Accuracy

\_\_/10

Educational Value

\_\_/10

Consistency

\_\_/10

Clarity

\_\_/10

Completeness

\_\_/10

Overall

\_\_/10

Scores below 8/10 require revision.

Scores below 7/10 should not be accepted.

---

# Repository-Wide Reviews

Every 5–10 completed chapters perform a repository review.

Check:

- duplicated explanations
- inconsistent terminology
- inconsistent diagrams
- chapter ordering
- pacing
- cross references
- repeated examples
- missing references
- architectural consistency

Repository reviews may suggest improvements across multiple chapters.

---

# Frozen Chapters

After acceptance:

- Commit to Git.
- Tag the chapter as frozen.

Frozen chapters should not be modified unless:

- factual error
- terminology update
- consistency correction
- editorial improvement affecting multiple chapters

Avoid rewriting frozen chapters.

---

# Versioning

Recommended Git workflow:

feature/chapter-01

↓

review/chapter-01

↓

main

Commit examples:

draft: complete chapter 01

review: address technical review

edit: copy editing chapter 01

final: freeze chapter 01

---

# AI Reviewer Rules

AI reviewers should:

- review rather than rewrite
- explain issues before proposing changes
- preserve the author's teaching style
- avoid introducing unnecessary complexity
- respect Book-Bible.md
- respect outline.md
- respect ADRs.md

The AI should never rewrite an entire chapter unless explicitly requested.

---

# Human Reviewer Rules

The human reviewer has the final authority.

When AI reviewers disagree:

1. Book-Bible.md takes precedence.
2. ADRs take precedence.
3. Outline.md takes precedence.
4. Human judgement takes precedence.

---

# Definition of Done

A chapter is complete when:

✓ All critical issues resolved.

✓ Major issues reviewed.

✓ Acceptance checklist passes.

✓ Quality score ≥ 8/10.

✓ Committed to Git.

✓ Marked as Frozen.

Only then may work begin on the next chapter.

---

End of REVIEW_GUIDELINES.md
