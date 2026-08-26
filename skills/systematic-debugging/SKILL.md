---
name: systematic-debugging
description: Use when investigating a concrete bug, test failure, build failure, integration problem, performance regression, or unexpected behavior. Find and verify the root cause before proposing or applying a fix.
---

# Systematic debugging

## Core rule

Do not fix symptoms before understanding the root cause.

A plausible explanation is not evidence. A successful-looking code change is not proof that the original problem is understood.

## Scope

Use this skill for a concrete observed problem, including:

- failing tests
- production or local bugs
- build and CI failures
- unexpected runtime behavior
- integration failures
- configuration or environment differences
- performance regressions

Do not use it merely to search for hypothetical bugs in otherwise working code; use a bug-hunt or code-review skill for that.

## Process

### 1. Establish the symptom

Before changing anything:

1. Read the complete error, stack trace, logs, exit code, and relevant warnings.
2. State exactly what is failing and what was expected instead.
3. Reproduce the problem when possible and record the minimal reproduction steps.
4. If it cannot be reproduced reliably, gather more evidence instead of guessing.

### 2. Check context and recent changes

Inspect only context that can plausibly affect the symptom:

- current diff
- recent relevant commits
- dependency changes
- configuration changes
- environment differences
- input data and state

Do not treat unrelated pre-existing failures as part of the current bug without evidence connecting them.

### 3. Trace the failure to its source

Follow the data and control flow backwards from the observed failure.

For each relevant boundary, determine:

- what enters
- what leaves
- which assumptions are made
- which configuration/state is used
- where the first incorrect value or behavior appears

In multi-component systems, inspect boundaries such as:

- controller -> service
- service -> repository
- app -> database
- producer -> queue -> consumer
- CI -> build script -> runtime
- service -> external API

Add temporary diagnostics only when they materially help locate the failing boundary. Do not leave unnecessary diagnostics in the final change.

### 4. Compare with a working pattern

When a similar working implementation exists in the same repository:

1. Find it.
2. Read enough of it to understand the whole relevant pattern.
3. Compare working and failing paths.
4. List meaningful differences before deciding which one matters.

Prefer repository conventions over inventing a new pattern.

### 5. Form one falsifiable hypothesis

State the hypothesis explicitly:

> I think X is the root cause because Y. If this is correct, Z should happen when we test it.

Test the smallest possible thing that can confirm or reject that hypothesis.

Change one variable at a time. Do not stack speculative fixes.

If the hypothesis is rejected, revert temporary changes where appropriate and form a new hypothesis from the new evidence.

### 6. Implement the smallest root-cause fix

After the root cause is supported by evidence:

- fix the source of the problem rather than a downstream symptom
- avoid unrelated refactoring
- avoid opportunistic cleanup
- add or update a focused regression test when practical and valuable
- preserve existing behavior outside the task scope

A regression test is strongly preferred for repeatable defects, but do not invent artificial tests when the failure is purely environmental or cannot reasonably be represented in the project test suite.

### 7. Verify

Verify the original symptom, not merely the changed code.

Run the narrowest useful verification first, then broader checks when appropriate.

Report separately:

- verification relevant to the current fix
- broader checks that passed
- unrelated pre-existing failures

Never modify unrelated production code or tests merely to make unrelated failures disappear.

## Failed attempts

After each failed fix attempt, return to evidence and reconsider the hypothesis.

If three materially different fix attempts have failed, stop applying more speculative changes. Re-evaluate whether the assumed architecture, ownership boundary, or problem statement is wrong before trying another fix.

## Output when investigating

Keep the result concise and evidence-based:

1. Symptom
2. Evidence
3. Root cause or current hypothesis
4. Minimal fix
5. Verification
6. Remaining uncertainty, if any

Do not claim a root cause when the evidence supports only a hypothesis.
