---
name: task-investigation
description: Use before implementing a task in an unfamiliar or partially understood codebase. Investigate the task's actual change surface, existing patterns, dependencies, tests, risks, and likely files before proposing implementation. Do not modify code during investigation.
---

# Task investigation

## Goal

Understand enough of the existing system to propose a grounded implementation plan without prematurely changing code.

This skill is especially useful when:

- the repository or service is unfamiliar
- the task wording is short or ambiguous
- the feature crosses several layers
- an external integration is involved
- the likely change surface is not yet known

## Hard rule

During investigation, do not modify source code, tests, configuration, migrations, generated files, or dependencies.

Reading commands and non-mutating diagnostics are allowed.

Do not fix unrelated problems discovered during investigation.

## Process

### 1. Parse the task

Extract:

- requested behavior
- explicit constraints
- acceptance criteria
- terminology and named components
- things the task does not ask to change

Separate facts from assumptions.

### 2. Find the domain and entry points

Locate the smallest set of entry points that can lead to the requested behavior, such as:

- REST/GraphQL controller or resolver
- command handler
- message/event consumer
- scheduled worker
- WebSocket gateway
- CLI command
- application service

Do not start by reading arbitrary files with matching words if a clearer entry point exists.

### 3. Build the relevant flow

Trace the current behavior through the system:

```text
entry point
   ↓
application/service layer
   ↓
domain/orchestration
   ↓
repository / queue / external adapter
   ↓
storage or external system
```

Record only relevant branches.

### 4. Inspect data and state

Determine whether the task touches:

- DTOs or schemas
- domain models/entities
- database tables/collections
- migrations
- caches
- queues/events
- configuration/environment variables
- external API contracts
- serialization/deserialization

### 5. Find existing patterns

Search for the closest working analogue in the same repository.

Prefer adapting established project conventions over introducing a new architecture.

Inspect enough of the analogue to understand:

- layering
- naming
- error handling
- validation
- transaction boundaries
- tests
- logging

### 6. Inspect relevant tests

Find tests that describe current behavior around the change surface.

Classify them as:

- directly relevant
- useful regression coverage
- unrelated

Do not treat an unrelated failing test as something to fix merely because it was discovered.

### 7. Inspect history selectively

Use git history/blame only when it can answer a concrete question, such as:

- why an unusual decision exists
- whether behavior was intentionally changed
- which files historically changed together

Do not browse history for its own sake.

### 8. Identify the likely change surface

List:

- files/components likely to change
- files/components likely not to change
- new files that may be needed
- contracts that must remain compatible

This is a prediction, not permission to edit.

### 9. Identify risks and unknowns

Call out concrete uncertainties that could alter the implementation, for example:

- unclear ownership of state
- concurrency/idempotency
- transaction boundaries
- backward compatibility
- external API behavior
- migration/data compatibility
- caching
- authorization
- event ordering

Do not manufacture generic risks that are not supported by the code or task.

### 10. Produce an implementation plan

The plan should be ordered by dependency and contain small coherent steps.

For each step, name the relevant component when known and state what behavior changes.

Include verification appropriate to the task.

## Unrelated failures policy

Existing failing tests, lint violations, legacy issues, flaky tests, environment-specific failures, and unrelated code smells are context only.

Do not:

- fix them
- refactor them
- change production behavior to satisfy them
- include them in the implementation scope

unless there is evidence that the requested task caused, depends on, or materially affects the failure.

If relevant, mention them briefly as pre-existing context and continue the investigation.

## Expected output

Produce a concise investigation report with:

1. Task understanding
2. Current relevant flow
3. Existing analogue/pattern
4. Likely change surface
5. Relevant tests
6. Risks/unknowns
7. Implementation plan
8. Verification plan

Do not present assumptions as facts.
