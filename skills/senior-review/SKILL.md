---
name: senior-review
description: Strict senior-level production code review. Use for deep pre-MR review, final review after implementation, or when ordinary code review reports CLEAN but stronger scrutiny is required. Focuses on behavioral correctness, dead code, duplication, conflicting responsibilities, double validation/normalization, object spread semantics, test intent, missing cases, weak assertions, unrealistic mocks, and maintainability.
---

# Senior Review

## Purpose

Perform a strict senior-level code review focused not only on obvious bugs, but also on maintainability, semantic correctness, redundancy, test quality, conflicting responsibilities, unnecessary complexity, and hidden regressions.

This review must approximate the level of scrutiny expected from an experienced senior engineer reviewing a production merge request.

The goal is **not** to confirm that the implementation looks reasonable.

The goal is to find the strongest legitimate reasons why the change may need revision.

Assume the implementation can contain subtle defects even when:

- the code compiles;
- lint passes;
- tests pass;
- the implementation looks clean;
- the diff is small;
- another review already returned `CLEAN`.

Do not invent speculative issues.

Every finding must be supported by concrete code, behavior, architecture, tests, or a plausible execution path.

---

# Core principles

## 1. Do not review the diff in isolation

A changed line can only be evaluated correctly in the context in which it executes.

Inspect the semantic neighborhood of the change where relevant:

- callers;
- callees;
- sibling methods;
- existing helpers;
- interfaces and contracts;
- DTOs and schemas;
- validators;
- normalizers;
- repositories;
- services;
- controllers;
- workers;
- adapters;
- related tests;
- similar implementations elsewhere in the repository.

Do not read the entire repository without reason.

Expand outward from the changed code only as far as necessary to understand its actual responsibility and execution path.

---

## 2. Reconstruct intent before judging implementation

Before looking for findings, determine:

1. What behavior is being introduced or changed?
2. What existing behavior must remain unchanged?
3. Which execution paths are affected?
4. Where should the responsibility logically live?
5. Which existing code already performs related work?
6. Which invariants must remain true?

Do not assume that the current implementation represents the intended behavior.

Treat tests and production code as separate sources of evidence rather than assuming either one is authoritative.

---

# Mandatory review pipeline

Perform every pass below.

Do not skip a pass merely because earlier passes found no issues.

---

## Pass 1 — Behavioral reconstruction

Describe internally:

- the requested or inferred behavior;
- affected input scenarios;
- expected outputs or side effects;
- important invariants;
- relevant failure modes.

Construct a small behavioral case matrix where applicable.

Consider at least:

- normal case;
- empty/missing input;
- invalid input;
- duplicate/repeated input;
- already-normalized/already-processed input;
- boundary conditions;
- failure/error path;
- repeated execution/idempotency where relevant.

Use repository context to determine which cases actually matter.

Do not mechanically invent irrelevant edge cases.

---

## Pass 2 — Correctness

Look for:

- incorrect conditions;
- inverted conditions;
- incorrect branching;
- missing branches;
- wrong fallback behavior;
- incorrect precedence;
- unintended mutations;
- stale values;
- incorrect assumptions about null/undefined;
- wrong async behavior;
- race conditions;
- lost errors;
- swallowed exceptions;
- incorrect error propagation;
- incorrect return values;
- wrong state transitions;
- incorrect persistence behavior;
- inconsistent behavior between equivalent paths.

Trace values through the actual execution flow where necessary.

---

## Pass 3 — Redundancy and accidental complexity

Explicitly search for:

- dead code;
- unreachable code;
- unused helpers;
- unused branches;
- redundant variables;
- unnecessary wrappers;
- unnecessary abstractions;
- duplicate conditions;
- duplicate guards;
- duplicate validation;
- duplicate normalization;
- repeated parsing;
- repeated serialization;
- repeated mapping;
- repeated filtering;
- repeated transformations;
- duplicate fallback logic;
- duplicate database lookups;
- duplicate external calls;
- duplicate state checks;
- redundant object reconstruction;
- redundant spreads;
- spread chains that obscure precedence;
- values transformed and then transformed again;
- values validated and then validated again;
- checks performed at multiple layers without a clear reason;
- multiple sources of truth.

Ask for every meaningful new line:

> Why does this line need to exist?

Ask for every meaningful transformation:

> Has this already happened earlier in the flow?

Ask for every validation:

> Is the same invariant already guaranteed by a previous boundary?

Ask for every helper:

> Does an equivalent helper already exist?

---

## Pass 4 — Responsibility conflicts

Look for methods, services, helpers, or layers whose responsibilities overlap.

Explicitly search for:

- two methods doing almost the same thing;
- two ways to obtain the same result;
- duplicated business rules;
- duplicated state transitions;
- duplicated normalization policies;
- competing sources of truth;
- logic split between layers without a clear boundary;
- service logic duplicated in controllers/workers/repositories;
- repository behavior leaking into domain logic;
- validation duplicated across DTO/schema/service layers without justification.

When two methods appear related, compare their contracts and behavior.

Ask:

- Why do both methods exist?
- Is one a partial duplicate of the other?
- Can their behavior diverge?
- Does one bypass rules enforced by the other?
- Which one is the authoritative path?

---

## Pass 5 — Data transformation audit

Follow important values from their origin to final consumption.

For each important value, check whether it is:

- parsed;
- normalized;
- trimmed;
- lowercased;
- converted;
- mapped;
- validated;
- defaulted;
- sanitized;
- serialized;
- deserialized;

more than once.

Pay special attention to:

- IDs;
- dates;
- timezones;
- emails;
- URLs;
- enum-like strings;
- optional values;
- arrays;
- participant/user identifiers;
- database values;
- external API data.

Double normalization or double validation is a review finding when it introduces unnecessary work, conflicting semantics, hidden assumptions, or unclear ownership.

Do not report harmless repetition solely for stylistic reasons.

---

## Pass 6 — Object spread and merge audit

Treat object spread as executable logic, not syntax sugar.

For every non-trivial object merge or spread chain, inspect:

- property precedence;
- overwritten values;
- unintended preservation of stale properties;
- optional properties;
- undefined values;
- defaults;
- duplicated transformation;
- mutation vs reconstruction;
- whether important fields can be silently replaced.

Example questions:

- Which object wins for each overlapping property?
- Can normalized data be overwritten by raw data?
- Can a default overwrite an explicit value?
- Can undefined erase a previously valid property?
- Is the spread hiding unnecessary reconstruction?

---

## Pass 7 — Test intent audit

Review tests independently from production implementation.

For every added or meaningfully changed test, determine:

1. What behavior does the test name claim to verify?
2. What behavior does the test actually verify?
3. Do the assertions prove the claimed behavior?
4. Is the test checking observable behavior or implementation details?
5. Is the setup representative of the real scenario?
6. Is the test testing the feature or merely making the current implementation green?

A test passing is not evidence that the test is useful.

---

## Pass 8 — Test mutation thought experiment

For every important test, ask:

> What realistic broken implementation would still make this test pass?

Consider realistic mutations such as:

- removing an important condition;
- reversing a condition;
- returning a constant;
- ignoring one input;
- removing normalization;
- performing normalization twice;
- skipping validation;
- using the wrong fallback;
- using stale state;
- calling the wrong dependency;
- not persisting a value;
- persisting the wrong value;
- accepting an invalid case;
- failing to reject an invalid case.

If an important realistic defect survives the test suite, report the missing assertion or missing scenario.

Do not demand exhaustive mutation testing.

Focus on bugs plausibly related to the change.

---

## Pass 9 — Test case completeness

Derive test cases from behavior, not from the current code branches alone.

Check relevant combinations of:

- positive case;
- negative case;
- missing value;
- invalid value;
- already-processed value;
- duplicate value;
- boundary value;
- dependency failure;
- partial data;
- repeated execution.

Check whether branch combinations matter.

Do not report every mathematically possible combination.

Report missing cases only when they represent materially different behavior or plausible regressions.

---

## Pass 10 — Test naming consistency

Compare each test name with:

- setup;
- action;
- assertions.

Report a mismatch when the title promises behavior that is not actually demonstrated.

Examples:

- name says "does not call X", but there is no assertion proving X was not called;
- name says "returns normalized value", but only status code is asserted;
- name says "handles missing timezone", but test data contains a timezone;
- name says "throws when invalid", but assertion accepts any rejection.

Test names are part of the contract.

---

## Pass 11 — Mock quality

Inspect mocks and stubs for tests affected by the change.

Look for:

- mocks that make the test impossible to fail;
- overly broad mocks;
- unrealistic responses;
- mocks reproducing the implementation;
- mocked method returning exactly the value the assertion expects without exercising meaningful logic;
- missing verification of calls where interactions are important;
- unnecessary mocks hiding integration between units.

Do not reject mocking merely because a test uses mocks.

The question is whether the mock preserves the behavior being tested.

---

## Pass 12 — Architecture and maintainability

Ask:

- Is this responsibility in the correct layer?
- Does the change introduce another path for the same operation?
- Is an existing abstraction being bypassed?
- Does this create a new source of truth?
- Is a simple operation being made unnecessarily indirect?
- Will the next developer know where this rule belongs?
- Does the API/contract still make semantic sense?
- Is the implementation consistent with nearby repository conventions?

Prefer repository consistency over abstract architectural purity unless the existing pattern is itself causing a concrete problem.

---

## Pass 13 — Deletion test

Mentally attempt to remove:

- new helpers;
- conditions;
- mappings;
- transformations;
- fields;
- branches;
- variables;
- wrappers.

If removal would not materially change intended behavior, investigate whether the code is unnecessary.

Do not report code as dead solely because its effect is not immediately obvious.

Trace usage first.

---

## Pass 14 — Adversarial review

Assume the code is subtly wrong even if everything looks clean.

Your task in this pass is to find the strongest legitimate reason an experienced reviewer could request changes.

Try to falsify the implementation.

Ask:

- What assumption is this code making?
- Can that assumption be false?
- What happens on the second execution?
- What happens with partially valid input?
- What happens when dependency output differs slightly from the happy path?
- Is this behavior enforced or merely implied?
- Could two related methods disagree?
- Could a test pass while production behavior is wrong?
- Could this branch be impossible or redundant?
- Is the implementation fixing the symptom rather than the responsibility boundary?

Do not invent hypothetical system requirements unsupported by the repository or task.

---

# NestJS / TypeScript specific checks

When applicable, inspect:

- controller/service/repository responsibility boundaries;
- DTO/schema validation ownership;
- pipes;
- guards;
- interceptors;
- exception mapping;
- dependency injection;
- provider scope;
- module registration;
- async error handling;
- Promise handling;
- nullable/optional TypeScript semantics;
- unsafe type assertions;
- type narrowing;
- object spread with optional properties;
- enum/string mismatch;
- date/time/timezone conversions;
- database model vs domain model transformations;
- transaction boundaries;
- repeated repository calls;
- worker/service duplicated logic;
- event handlers and idempotency.

Do not emit generic NestJS best-practice comments unrelated to the change.

---

# Evidence requirements

Every finding must include enough evidence to make it actionable.

A finding should explain:

- where the problem is;
- what is wrong;
- why it matters;
- what execution path or test demonstrates the concern;
- what category it belongs to.

Prefer concrete statements such as:

> `normalizeEmail()` is called in `A` before `B`, while `B` calls it again internally. The value is therefore normalized twice on this path.

Avoid vague statements such as:

> This could possibly be simplified.

---

# Severity

Use severity based on actual impact.

## BLOCKER

Use only when the change should clearly not merge.

Examples:

- data corruption;
- serious security issue;
- destructive behavior;
- fundamentally incorrect implementation.

## HIGH

Likely production defect or substantial behavioral regression.

## MEDIUM

Real maintainability, correctness-risk, or test-quality issue that should normally be fixed before merge.

Examples:

- duplicated business rule;
- conflicting normalization ownership;
- meaningful missing test scenario;
- test that does not verify its claimed behavior;
- overlapping methods likely to diverge.

## LOW

Minor but legitimate improvement.

Do not flood the review with LOW findings.

---

# Anti-noise rules

Do not report:

- personal style preferences;
- arbitrary renaming;
- formatting already handled by tooling;
- generic "could be cleaner" comments;
- theoretical abstractions with no concrete benefit;
- speculative edge cases unsupported by domain context;
- unrelated pre-existing issues;
- issues outside the changed semantic area unless the change directly interacts with them.

Do not manufacture findings to avoid returning CLEAN.

A strict review must still be truthful.

---

# Existing unrelated failures

Do not modify production code or tests merely to fix unrelated pre-existing test failures.

If tests unrelated to the reviewed change already fail in the local/development environment but are known to pass in CI or are outside the task scope:

- mention them only when they prevent verification;
- do not treat them as findings against the change;
- do not propose changing unrelated code to make them green.

---

# CLEAN gate

`CLEAN` is a strong conclusion.

Do not return `CLEAN` merely because no obvious bug was found.

Before returning `CLEAN`, verify all applicable items:

- [ ] Intended behavior was reconstructed.
- [ ] Relevant callers and callees were inspected.
- [ ] Relevant sibling implementations were compared.
- [ ] No dead or obviously unnecessary code was found.
- [ ] No duplicated business rules were found.
- [ ] No duplicated validation was found without justification.
- [ ] No duplicated normalization was found without justification.
- [ ] No conflicting or overlapping method responsibilities were found.
- [ ] No multiple sources of truth were introduced.
- [ ] Non-trivial object spreads/merges were checked for precedence problems.
- [ ] Important value transformations were traced through the execution path.
- [ ] Tests were reviewed independently from implementation.
- [ ] Test names match what the tests actually demonstrate.
- [ ] Important behavioral cases are covered.
- [ ] Important negative/failure cases are covered where relevant.
- [ ] No obvious realistic broken implementation would survive the important tests.
- [ ] Mocks do not trivially guarantee test success.
- [ ] The implementation is consistent with relevant repository architecture.
- [ ] No meaningful issue from the adversarial pass remains.

If an applicable item could not be verified because required context is unavailable, do not claim full `CLEAN`.

Instead state the verification limitation.

---

# Output format

Return findings first, ordered by severity.

For every finding use:

## [SEVERITY] Short finding title

**Location:** file / method / test

**Problem:**  
Concrete description of the issue.

**Why it matters:**  
Behavioral, architectural, maintainability, or test-quality consequence.

**Evidence:**  
Relevant execution path, duplicated responsibility, missing assertion, surviving mutation, or other concrete evidence.

**Recommended direction:**  
Describe the intended direction without unnecessarily rewriting the implementation.

---

After findings include:

## Review coverage

Briefly state which relevant areas were inspected:

- production execution path;
- surrounding methods;
- redundancy;
- transformations;
- tests;
- mocks;
- behavioral cases;
- architecture.

Do not dump internal reasoning.

---

If no findings remain, return:

# CLEAN

No actionable findings were identified after completing the senior review pipeline.

Then briefly state the review coverage.

Do not output `CLEAN` if relevant portions of the change could not be meaningfully inspected.

---

# Review behavior

Be skeptical but evidence-driven.

Prefer one strong finding over five speculative findings.

Do not optimize for politeness.

Do not optimize for finding something at all costs.

Optimize for catching the kinds of issues an experienced senior reviewer would reasonably block or comment on before merge.

Compilation, lint success, passing tests, and previous reviews are evidence, not proof of correctness.
