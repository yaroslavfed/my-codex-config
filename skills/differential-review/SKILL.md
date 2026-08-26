---
name: differential-review
description: Use for security-focused review of a branch, PR, commit range, or diff. Prioritize security-sensitive changes, compare against the baseline and relevant git history, assess blast radius and test coverage, and report evidence-backed regression risks.
disable-model-invocation: true
---

# Differential security review

Review the change, not the whole repository, with emphasis on security regressions introduced by the delta.

Use standard code review for purely cosmetic or documentation-only changes.

## Principles

- Risk first: prioritize auth, authorization, validation, secrets, external calls, deserialization, file/network access, state transitions, and sensitive data.
- Baseline aware: understand what behavior existed before the change.
- Evidence based: tie findings to specific changed code, callers, history, tests, or concrete attack/failure scenarios.
- Honest coverage: state what was and was not inspected.

## Workflow

### 1. Establish the diff

Determine the intended baseline and inspect the complete relevant diff.

Classify changed files/areas:

- high risk: auth/authz, validation removal, external calls, secrets, crypto, trust boundaries, dangerous serialization, privilege changes
- medium risk: business state changes, persistence behavior, public contracts, concurrency/idempotency
- low risk: comments, formatting, non-behavioral tests/logging

Depth of review follows risk, not line count.

### 2. Recover baseline intent

For security-relevant removed or weakened behavior, inspect relevant history/blame when useful.

Ask:

- what invariant existed before?
- why was this check added?
- is a previous security or bug fix being undone?

Do not browse history indiscriminately.

### 3. Analyze changed trust boundaries

For each high/medium-risk change inspect:

- who controls the input
- validation and normalization
- authorization before side effects
- sensitive data exposure
- error/fail-open behavior
- external call assumptions
- persistence/state invariants
- concurrency/idempotency where relevant

### 4. Assess blast radius

Find important callers, consumers, inherited implementations, event flows, or public endpoints affected by the changed contract.

Do not assume the changed file is the full impact surface.

### 5. Check regression coverage

Determine whether tests exercise the security-relevant invariant being changed.

Missing tests are a risk signal, not proof of a vulnerability.

### 6. Adversarial check

For plausible findings, construct a concrete scenario:

1. attacker or malformed-input capability
2. entry point
3. changed behavior
4. violated invariant
5. resulting impact

Reject generic findings that cannot be connected to a credible path.

## Finding quality bar

A finding should contain:

- severity/confidence
- changed location
- baseline behavior
- regression/problem
- concrete scenario or failure mode
- blast radius
- recommended remediation

Do not inflate severity because a category sounds scary.

## Output

Default to a concise chat report unless the user explicitly asks for a markdown report file.

Structure:

1. Scope and baseline
2. High-risk changed areas
3. Findings ordered by severity
4. Test/coverage gaps relevant to security
5. Coverage limitations

If there are no meaningful findings, say so and still note the security-sensitive areas reviewed.
