---
name: verification-before-completion
description: Use before claiming that implementation work is complete, fixed, correct, passing, or ready. Require fresh evidence from relevant checks and distinguish task-related failures from unrelated pre-existing failures.
---

# Verification before completion

## Core rule

Do not claim success without fresh evidence that directly supports the claim.

"Should work", "looks correct", and "the code compiles in my head" are not verification.

## Before saying the task is complete

### 1. Define what would prove completion

Derive verification from the actual task and changed surface.

Possible checks include:

- targeted unit or integration tests
- relevant e2e tests
- type checking
- build
- lint
- database migration validation
- API/manual reproduction
- runtime logs
- generated artifacts or schemas

Do not run every available check mechanically. Prefer checks that can actually falsify the implementation.

### 2. Run relevant checks now

Use fresh output from the current state of the working tree.

Do not rely on:

- a previous run before the final change
- another person's report
- assumptions based on code inspection alone

### 3. Read the result, including exit status

Do not summarize a command as successful unless its actual output and exit status support that conclusion.

If output is truncated or ambiguous, say so and obtain a clearer result when possible.

### 4. Separate related and unrelated failures

A repository may already contain failing tests, lint violations, flaky tests, environment-specific failures, or legacy problems unrelated to the current task.

Treat them as context, not automatic blockers, when all of the following are true:

- they are outside the task scope
- the current change does not touch or materially affect them
- evidence indicates they were pre-existing or environment-specific
- task-relevant verification passes

In that case report them explicitly, for example:

- relevant tests: pass
- build: pass
- two unrelated e2e tests: fail locally as before; not modified

Do not change production code, unrelated tests, or expected behavior merely to make unrelated checks green.

If there is evidence that the current change caused or influences the failure, it is not unrelated and must be investigated.

### 5. Match claims to evidence

Allowed:

> Targeted tests pass and the project builds successfully. Two known unrelated e2e tests still fail locally.

Not allowed:

> Everything passes.

when everything was not actually checked or known unrelated failures remain.

### 6. If verification cannot be completed

Do not disguise the limitation.

State:

- what was verified
- what could not be verified
- why
- what risk remains

A partially verified implementation can still be useful; an unsupported claim of completion is not.

## Final completion summary

When appropriate, finish with a short evidence-oriented summary:

- Changed: what was implemented
- Verified: commands/checks and outcome
- Not verified: anything unavailable
- Existing unrelated failures: only if present
