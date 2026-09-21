---
name: senior-review
description: Strict senior-level production pre-MR review. Use after implementation or before merge when ordinary review is insufficient. Reconstructs behavior and entity lifecycles, enumerates every production writer/creator path, traces contracts across repositories and services, proves branch reachability, attacks authorization and ownership transitions, challenges tests with mutation thought experiments, searches repository-wide for duplicated helpers/constants/predicates and deleted coverage, evaluates storage-model complexity, DB/I/O cost, shared test state, migrations, configuration, observability, and cleanup. Findings are classified by evidence and scope; not every finding implies an in-scope fix.
---

# Senior Review

## Purpose

Perform a strict production-grade review that tries to discover the kinds of defects an experienced human reviewer would find after reading beyond the diff.

The goal is not to confirm that the implementation looks reasonable.

The goal is to falsify it:

- reconstruct what the change is supposed to do;
- trace the real production execution paths;
- find contradictions between related paths;
- prove that new branches and defensive checks are actually reachable;
- challenge tests with realistic mutations;
- inspect the cost and side effects of repeated execution;
- enumerate every production path that can create, import, migrate, restore, or mutate the affected entity;
- search the repository for existing implementations before accepting a new helper, constant, filter, bridge, transaction wrapper, registry, or abstraction;
- challenge whether new persistence structures are actually needed or merely encode fields that belong to an existing aggregate;
- compare removed tests against newly added coverage instead of assuming test count or green CI preserves behavior;
- distinguish a real defect from a speculative improvement or unrelated debt.

Assume subtle defects may exist even when:

- code compiles;
- lint passes;
- tests pass;
- the diff is small;
- the implementation looks clean;
- another review returned `CLEAN`.

Do not invent issues. Every finding must have concrete evidence from code, contracts, tests, execution paths, persistence behavior, or a realistic production scenario.

---

# Core review model

The mandatory review order is:

1. **Behavior and lifecycle**
2. **Contracts and persistence**
3. **Security / ownership transitions**
4. **Reachability**
5. **Repository-wide consistency**
6. **Adversarial tests**
7. **Cost, architecture, and side effects**
8. **Cleanup and operability**

Do not skip later passes because earlier ones look clean.

---

# Pass 1 — Reconstruct behavior before judging code

Internally determine:

- what behavior is introduced, removed, or changed;
- what existing behavior must remain unchanged;
- which actors and execution paths are affected;
- what side effects can occur;
- what invariants must remain true;
- what failures are acceptable;
- what failures must be visible or fatal;
- every way the affected entity/state can come into existence.

For each important entity, explicitly enumerate lifecycle entrypoints when applicable:

`API create -> admin create -> import/script -> migration/backfill -> bootstrap/seed -> sync/reconciliation -> restore/legacy data -> direct repository writer`

Do not assume the method changed in the diff is the only creator.

If the implementation relies on an invariant such as "every bot has three configuration rows", prove which component establishes that invariant for **every** lifecycle entrypoint and when it becomes true.

Build a small case matrix when useful.

Consider only domain-relevant cases, for example:

- normal input;
- empty input;
- `null`;
- `undefined`;
- empty string;
- whitespace-only string;
- stale persisted data;
- duplicate input;
- repeated execution;
- partial external payload;
- invalid payload;
- dependency failure;
- boundary time/date case;
- multi-user or multi-consumer case.

Do not mechanically enumerate irrelevant edge cases.

Tests and production code are independent evidence. Neither is automatically the specification.

---

# Pass 2 — Expand beyond the diff

Do not review a changed line in isolation.

Inspect the semantic neighborhood where relevant:

- callers;
- callees;
- sibling methods;
- interfaces;
- DTOs and schemas;
- validators;
- parsers and normalizers;
- repositories;
- models;
- workers;
- cron jobs;
- lifecycle hooks;
- commands;
- guards;
- templates/formatters;
- migrations;
- env/config files;
- docs;
- tests;
- similar implementations elsewhere.

Expand only as far as necessary to understand real behavior.

For a changed method, use usages/call hierarchy when available instead of relying only on text search.

For every newly introduced helper, constant, predicate, registry, adapter, transaction wrapper, stream bridge, or generic-looking utility, perform a repository-wide search for:

- the same symbol name;
- equivalent literals;
- equivalent query/filter shapes;
- equivalent call patterns;
- sibling implementations with slightly different names.

Do not conclude "new helper is fine" before checking whether the repository already has the abstraction or business rule.

Ask:

> If this value is wrong here, what is the furthest downstream consequence?

---

# Pass 3 — Contract flow audit

For every important value changed by the branch, trace:

`external input -> parser/validator -> canonical representation -> persistence -> readers -> comparisons -> output/side effects`

Pay special attention to:

- IDs;
- foreign/external IDs;
- department/user identifiers;
- dates;
- timezones;
- email/URL values;
- enum-like strings;
- nullable fields;
- optional fields;
- arrays;
- snapshots;
- persisted legacy values.

Determine explicitly:

- where normalization belongs;
- whether the internal representation is canonical;
- whether `null`, `undefined`, `''`, and whitespace mean different things;
- whether all writers produce the same form;
- whether all readers expect the same form;
- what a missing persisted record means;
- whether absence is a valid default state, a legacy state, or corruption;
- whether readers can distinguish "not initialized yet" from "invalid data";
- whether a migration/backfill is required for correctness or merely compensates for an avoidable storage design.

### Normalization ownership rule

Normalization should normally have one owner.

If a boundary guarantees canonical `string | null`, downstream code should not repeatedly call `trim()`, re-default, or reinterpret the value without a specific reason.

Flag cases where:

- one writer trims and another does not;
- a parser normalizes but a formatter normalizes again;
- fresh data is canonical but stale DB data is not accounted for;
- two readers disagree about nullable/optional semantics;
- the same field has multiple hidden contracts.

Do not report harmless formatting repetition as a defect unless it creates conflicting semantics, hidden assumptions, or maintainability risk.

### Mandatory writer inventory

For every changed persisted concept, build a compact internal table:

`writer/path | preconditions | value written | transaction | can record be absent? | readers affected`

Include imports, scripts, migrations, seeders, bootstrap code, repository calls, and legacy data paths.

A storage invariant is not proven until every writer/creator path has been checked.

Especially flag:

- a new side table/collection initialized only by one create path;
- imports or migrations that can produce the parent entity without dependent records;
- a reader that treats missing optional/default state as corruption;
- an invariant established asynchronously after the entity becomes externally visible.

---

# Pass 4 — Semantic symmetry audit

Compare neighboring paths that conceptually implement the same rule.

Examples:

- manual command vs nightly synchronization;
- cron vs interactive command;
- create vs update;
- create vs import vs migration/backfill vs bootstrap;
- freshly created entity vs legacy/imported entity;
- reminder vs immediate notification;
- API vs worker;
- admin path vs ordinary user path;
- snapshot writer vs cleanup reader;
- two consumers of the same external payload;
- two representations of the same user capability.

Ask:

- Do they define the same concept the same way?
- Can one path undo another?
- Does one keep stale state while another clears it?
- Do they use the same timezone, normalization, fallback, authorization scope, and identity rules?
- Does one path expose a capability another background process revokes?

A contradiction between two valid production paths is usually more important than local code style.

---

# Pass 5 — Reachability proof

For every new meaningful branch, fallback, guard, or defensive check, ask:

> Can this state actually occur through a production path?

Trace the state from real boundaries and invariants.

Particularly inspect:

- `if` branches;
- `??` fallbacks;
- optional arguments;
- impossible combinations of derived fields;
- cycle protection on JSON-derived structures;
- checks duplicated by an earlier invariant;
- branches that only tests can construct.

### Reachability rule

A unit test does not prove a state is realistic.

If the production model guarantees:

`B = f(A)`

then a test that sets `A` and `B` independently may create an impossible state.

When a branch only protects an impossible state:

- prefer removing the branch;
- or document the real invariant;
- or prove the production scenario that requires it.

Do not keep unreachable code merely because it can be unit-tested.

---

# Pass 6 — Correctness and async control flow

Check for:

- wrong conditions;
- inverted conditions;
- missing branches;
- stale state;
- accidental broadening/narrowing of access;
- incorrect fallbacks;
- unintended mutations;
- incorrect state transitions;
- partial side effects;
- swallowed errors;
- incorrect error propagation;
- races;
- missing `await`;
- `Promise` used as a boolean;
- async work started but not observed;
- work performed outside the intended transaction or try/catch;
- state mutated before authorization/ownership checks complete;
- a failed operation leaving durable partial state;
- a read-check-write sequence where the check observes state that the same operation just created.

For mutations that can fail after a write, explicitly ask:

> If the method throws on the next line, what has already been persisted?

For async boolean checks especially verify that the resolved boolean, not the Promise object, participates in the condition.

---

# Pass 7 — Duplicated business knowledge

Differentiate ordinary duplication from duplicated business rules.

Prioritize duplicated knowledge such as:

- timezone rules;
- authorization rules;
- normalization rules;
- relation/policy logic;
- event formatting semantics;
- composite key construction;
- fallback text;
- state transition rules;
- filtering rules;
- date windows;
- identity resolution.

Ask:

> If one copy changes next month, can the other silently remain stale?

Duplicated business knowledge is a stronger finding than duplicated syntax.

Do not demand abstraction for every repeated line. Prefer repository consistency and concrete divergence risk over abstract DRY purity.

### Repository-wide duplication search

When the branch adds any of the following, search the entire repository before approving it:

- constants and magic numbers;
- Mongo/SQL filters;
- authorization predicates;
- system-user/system-bot registries;
- stream/iterator adapters;
- transaction helpers;
- batch sizes;
- duplicate-key codes;
- retry/backoff values;
- identity predicates;
- mapping helpers.

Search both by symbol and by literal/structural shape. Different names do not mean different concepts.

If copies have already diverged, report the concrete divergence, not only "DRY violation".

---

# Pass 8 — Redundancy and deletion test

Explicitly inspect:

- dead code;
- unused helpers;
- unused public methods;
- duplicate conditions;
- duplicate guards;
- repeated transformations;
- no-op `?? null`;
- unnecessary ternaries;
- redundant object reconstruction;
- unused variables;
- repeated parsing;
- repeated queries;
- duplicate validation;
- duplicated predicates;
- stale comments;
- dead docs/config.

For every meaningful new line ask:

> Why does this line need to exist?

Mentally attempt to remove:

- the new branch;
- the new helper;
- the new fallback;
- a field from an update;
- a validation layer;
- a wrapper;
- a transformation.

If intended behavior remains unchanged, investigate whether the code is unnecessary.

---

# Pass 9 — Database and I/O cost audit

Inspect production paths for repeated work.

Look for:

- N+1 repository calls;
- repeated loads of the same entity;
- DB calls inside nested loops;
- external calls inside hot loops;
- the same query executed twice with identical arguments;
- broad reads followed by narrow in-memory filtering;
- expensive work before cheap dedup/guard checks.

### Order-of-work rule

When semantics allow, prefer:

`cheap rejection/dedup -> cache lookup -> DB/external I/O -> expensive transformation`

Flag code that pays for I/O before discovering the result will be discarded.

### Pass-local memoization

For repeated reads that must remain fresh between runs but not between iterations of one run, consider caching by stable key for the lifetime of the operation.

Do not recommend long-lived caching when freshness requirements are unclear.

---

# Pass 10 — Write amplification and repeated execution

For every write in cron, sync, worker, reconciliation, polling, or frequently called path ask:

> How many rows and how often?

Mentally multiply:

`writes per execution × executions per hour/day × entities`

Check for:

- blanket `UPDATE`;
- unchanged values rewritten every run;
- per-row updates replacing an existing bulk operation;
- large transactions around long loops;
- unnecessary write locks;
- writes occurring before change detection.

When applicable, ensure comparisons handle `NULL` correctly.

A write that is harmless once may be unacceptable every minute across thousands of rows.

---

# Pass 11 — Transaction semantics

Inspect:

- transaction boundaries;
- lock duration;
- lock type;
- hidden `FOR UPDATE`;
- external calls inside transactions;
- loops inside a single transaction;
- nested transaction semantics;
- optional transaction handling;
- repository methods whose names hide locking behavior.

A normal-looking repository method should not silently become a locking read merely because a transaction argument was passed, unless that convention is explicit and established.

Prefer names/contracts that expose important lock semantics.

Potential performance cost alone is not automatically a merge-blocking defect; classify it based on evidence.

---

# Pass 12 — Test intent audit

For every added or changed test determine:

1. What behavior does the name claim?
2. What setup actually reaches?
3. What assertions actually prove?
4. Could the assertion match an earlier unrelated call?
5. Is the test validating observable behavior or only implementation details?
6. Does the test use the real function whose correctness matters?
7. Does the fixture represent a reachable production state?

A passing test is not evidence that the test is useful.

### Deleted / moved test coverage

When production logic moves between services/files or tests are deleted:

1. identify tests removed from the old location;
2. map each removed behavioral claim to a test in the new location;
3. verify the replacement fails when the corresponding production rule is removed;
4. do not accept "covered elsewhere" without locating the exact assertion.

A moved implementation with fewer semantic tests is a regression in review coverage even if the suite remains green.

---

# Pass 13 — Mandatory mutation thought experiment

For every significant behavior, ask what minimal realistic mutation should make its test fail.

Try relevant mutations such as:

1. Remove the new condition.
2. Negate the condition.
3. Return a superset.
4. Return a subset.
5. Remove one important field from `where`.
6. Remove one important field from `update`.
7. Remove normalization.
8. Restore raw whitespace/empty-string behavior.
9. Omit `await`.
10. Return a constant.
11. Skip persistence.
12. Call the dependency one extra time.
13. Remove an output line the test name claims to cover.
14. Make two users/consumers receive the same entity.
15. Execute the scenario twice.
16. Run the test before/after another suite.
17. Break the real implementation hidden behind a mock.
18. Remove a type/ownership/authorization discriminator from an existing-entity path.
19. Make a dependent configuration record absent while the parent entity exists.
20. Route creation through import/bootstrap/migration instead of the normal API.
21. Delete one source-of-truth entry or add an unrecognized one.
22. Make the operation throw immediately after its first persistence call.

If the test still passes under a realistic mutation related to its claimed behavior, report the surviving mutation and the missing assertion/scenario.

Do not demand exhaustive mutation testing.

---

# Pass 14 — Assertion strength

Pay special attention to:

- `arrayContaining`;
- `objectContaining`;
- `toHaveBeenCalledWith`;
- `toThrow()` without expected reason;
- broad snapshots;
- assertions that only prove a subset;
- assertions that match any call instead of the relevant call;
- names that promise more than assertions verify.

Examples of suspicious patterns:

- exact result matters but `arrayContaining` accepts extras;
- transaction is asserted but critical `where` is not;
- a call occurs twice but the test checks only that it occurred at least once;
- several invalid cases all fail at the same outer guard;
- the test name mentions end time/location/authorization but the assertion omits it.

When order or occurrence matters, consider exact counts, `NthCalledWith`, exact collections, explicit negative assertions, or more observable behavior.

---

# Pass 15 — Mock integrity

Inspect mocks and helpers for false confidence.

Look for:

- mocking the exact pure function the integration scenario is supposed to verify;
- reimplementing production recursion/logic in test code;
- stubs that always return the expected assertion value;
- unrealistic combinations of derived fields;
- helper defaults that hide required production arguments;
- mocks that bypass parsers, normalizers, guards, or repositories relevant to the change.

Ask:

> If the real implementation is broken, can this test remain green because the test replaced it?

If yes, the test may be exercising choreography instead of behavior.

---

# Pass 16 — Shared integration/e2e state

For integration and e2e tests inspect lifecycle and isolation:

- `beforeAll`;
- `beforeEach`;
- `afterEach`;
- `afterAll`;
- global setup;
- shared DB/schema;
- shared Redis/queue state;
- migrations;
- destructive cleanup.

Treat these patterns as high-risk until justified:

- `destroy({ where: {} })`;
- `truncate`;
- `migration.down()`;
- dropping columns/tables;
- deleting all rows;
- changing shared config/state.

Ask:

- Does the suite restore schema/state?
- Does cleanup remove only records owned by this test?
- Could test order change the result?
- Does Jest cache/timing order mask the issue?
- Would this pass on a clean CI runner?

Tests must not leave the shared environment in a state that breaks later suites.

---

# Pass 17 — Migration audit

Review migrations bidirectionally.

For `up`:

- intended schema/data change;
- compatibility with existing rows;
- defaults;
- nullable semantics;
- indexes/constraints;
- locking or large-table impact when relevant.

For `down`:

- compare against the actual schema before `up`;
- restore original nullability/default/index/constraint behavior;
- do not invent a different historical schema.

If a migration test runs `down()` against a shared e2e DB, verify the test restores the schema afterwards or uses an isolated schema/database.

Also ask whether the migration is compensating for a persistence model that unnecessarily requires eager initialization.

If correctness requires a backfill solely because configuration was split away from an existing aggregate, compare that design against storing optional/default fields on the aggregate itself. Do not require redesign automatically, but report avoidable migration/invariant complexity when the simpler model is consistent with repository conventions.

For migration tests, verify the assertion observes a behavior the migration could actually violate. Assertions against mocks that never possessed the tested method/property are tautological and provide no protection.

---

# Pass 18 — Configuration lifecycle audit

For each new or changed configuration value inspect:

- code reader;
- `get` vs `getOrThrow`;
- defaults;
- env validation;
- committed env/example env;
- Docker/Compose;
- deployment manifests when present;
- tests;
- docs.

Classify the setting:

- required;
- optional with meaningful default;
- optional nullable.

### Fail-fast rule

If application correctness, access control, destructive behavior, external integration identity, or important user behavior depends on the value, prefer explicit required configuration and startup failure over a silent placeholder default.

Do not introduce fake production identities as fallback values for mandatory settings.

---

# Pass 19 — Feature deletion and cleanup

When code/integration/functionality is removed, inspect its entire footprint:

- imports;
- providers;
- dependencies;
- packages;
- types;
- env variables;
- credentials;
- secrets;
- Docker config;
- docs;
- tests;
- migrations;
- examples;
- comments.

Removing code but leaving dead secrets/config/docs is incomplete cleanup.

---

# Pass 20 — Observability and diagnosability

A safe fallback can still be operationally dangerous if nobody can see that it happened.

Inspect:

- warnings on degraded behavior;
- error messages;
- exception context;
- IDs necessary to diagnose the failure;
- log placement;
- retries misclassified as another failure;
- swallowed dependency failures.

If a global handler logs only `error.message`, a bare exception such as `UnauthorizedException()` may be insufficient when several branches can throw it.

Do not require logging for every minor fallback. Require it when behavior materially degrades, data is skipped, access changes, or support/debugging would otherwise be blind.

---

# Pass 21 — User-visible behavior and formatting consistency

When the change affects text, dates, timezones, templates, or notifications inspect the actual rendered output.

Check:

- all-day vs timed events;
- date format consistency;
- timezone labels;
- fallback wording;
- punctuation;
- stale technical labels;
- equivalent messages from different paths;
- template conditions that accidentally broaden visibility.

Do not assume reuse of a general formatter is always correct if the surrounding phrase has different semantics.

Evaluate the final user-visible sentence, not only the helper output.

---

# Pass 22 — Authorization and capability radius

When role/capability/department/ownership logic changes, trace every consumer.

Check:

- authorization guards;
- manual admin commands;
- background synchronization;
- impersonation;
- scope boundaries;
- subtree vs direct-parent semantics;
- docs;
- templates;
- cleanup/reconciliation.

Ask:

> Does changing this one boolean grant or revoke more capabilities elsewhere?

Treat broad capability radius as a reason to inspect all usages before approving the change.

### Ownership transition audit

For create-or-complete, claim, attach, import, activation, and "existing entity" flows, trace state in this order:

`state before call -> first write -> authorization/ownership check -> later writes -> externally visible result`

Verify that:

- authorization is based on pre-existing trusted state, not state just initialized from caller input;
- an unowned/legacy/imported entity cannot be claimed merely by supplying an owner identifier;
- insert-only defaults do not turn attacker-controlled input into authoritative ownership;
- forbidden operations leave no durable mutation;
- owner/system identity rules are enforced before secrets, webhooks, permissions, or external side effects are changed.

A check performed after the operation has established the condition it checks is not a valid security check.

---

# Pass 23 — Object spread and merge audit

Treat spreads as executable merge logic.

Inspect:

- precedence;
- raw values overwriting normalized values;
- `undefined` erasing valid values;
- stale fields being preserved;
- default values overriding explicit values;
- unnecessary object rebuilding;
- confusing conditional spreads.

Do not flag harmless spreads merely for style.

---

# Pass 24 — NestJS / TypeScript specific checks

When applicable inspect:

- DI wiring;
- modules and exports;
- provider scope;
- lifecycle hooks;
- `onModuleInit` startup failure radius;
- guards/interceptors;
- exception mapping;
- missing `await`;
- nullable/optional TypeScript semantics;
- unsafe assertions;
- DTO/runtime validation mismatch;
- repository/service responsibility;
- transaction boundaries;
- worker idempotency;
- cron overlap;
- repeated repository calls;
- one domain capability split across old and new services without a clear boundary;
- controllers depending on multiple services for one aggregate because behavior was only partially moved;
- loss of an existing assembler/mapper causing rereads after create/get/update;
- repository methods added only to support an avoidable transaction/storage shape.

Use the dedicated `nestjs-review` skill for a deeper NestJS-specific pass when relevant.

---


# Pass 25 — Entity lifecycle completeness

This is a mandatory pass for entities whose persistence shape or initialization changes.

Enumerate all ways the entity can appear in production:

- public API;
- internal API/gRPC;
- import scripts;
- CLI/admin scripts;
- migrations/backfills;
- seeds/bootstrap;
- synchronization;
- restore/replay;
- legacy rows/documents already present;
- tests/fixtures only as secondary evidence.

For each path answer:

- Does it create the same dependent state?
- Does it establish ownership the same way?
- Does it use the same defaults?
- Is the entity visible before initialization completes?
- Can readers observe a partially initialized entity?
- What happens if deployment skips an intermediate version?

### Upgrade-path rule

Always consider supported upgrade paths, not only "old version -> immediately previous version -> new version".

If a data invariant depends on a migration from an intermediate release, test or reason about installations that jump directly from an older release to the target release.

A valid historical entity must not become corrupt merely because it missed application-level initialization that did not exist when it was created.

---

# Pass 26 — Aggregate and persistence-model challenge

When the branch introduces a new collection/table/configuration store for data that belongs to an existing entity, challenge the model before reviewing only its implementation.

Ask:

- Does the parent entity already provide the natural identity and lifecycle?
- Are these values one-to-one with the parent?
- Are they always read together with the parent?
- Does the new store require eager default rows/documents?
- Does absence require special error handling or migrations?
- Does the split introduce transactions solely to keep two records consistent?
- Does it create multiple encodings of "empty"?
- Does it add repositories/services that are mostly pass-through?
- Does the repository already store similar fields directly on the aggregate?

Compare:

`fields/subdocument on existing aggregate`

against

`separate EAV/config table/collection`

Do not prescribe one universally. Report the architectural cost when the new model creates invariants, migrations, transactions, failure modes, or hundreds of lines of plumbing without an observed independent lifecycle or query requirement.

### Default-by-absence rule

For optional configuration, consider whether missing data can safely mean the domain default.

Do not automatically treat an absent configuration record as corruption if the business state can be represented without materializing a row/document.

---

# Pass 27 — Source-of-truth audit

For any registry, allowlist, denylist, capability list, system identity list, enum mirror, or protocol constant:

1. locate every definition in the current repository;
2. search sibling repositories/modules when the branch explicitly mirrors another component's contract and those sources are available;
3. identify the authoritative owner;
4. verify each copied entry is actually provisioned/created somewhere;
5. verify consumers agree on semantics.

Flag:

- two independent lists controlling authorization;
- an entry that exists only in the new branch;
- comments/docs naming a different source of truth;
- a local list that can silently drift from the provisioning system.

For security-relevant registries, duplicated ownership of truth is a correctness issue, not merely cleanup.

---

# Pass 28 — Responsibility and move-completeness audit

When logic is moved from one service/class/module to another, review the **move as a unit**, not only the new method.

Map related operations for the same aggregate:

`create | get | update | disable/delete | assemble/map | validate | authorize`

Ask:

- Did only half of one responsibility move?
- Does the controller now coordinate two services for one domain capability?
- Did the move lose access to an existing mapper/assembler and introduce rereads?
- Are there now duplicate validation/identity rules across both services?
- Would completing or reverting the move make the boundary clearer and cheaper?

Do not demand large refactors for aesthetics. Report split responsibility when it causes concrete duplication, extra I/O, inconsistent rules, or awkward orchestration introduced by the MR.

---

# Pass 29 — Partial side-effect and failure-atomicity audit

For every mutating request, write the side effects in exact order:

`read -> write A -> external call -> check -> write B -> response`

Then inject failure after each step.

Verify:

- unauthorized/conflicting requests do not persist state;
- a later `INTERNAL`, `ALREADY_EXISTS`, `FORBIDDEN`, or validation error does not leave earlier writes behind;
- related writes are either in one transaction or safely idempotent/recoverable;
- methods named as one logical operation are failure-atomic from the caller's perspective when required.

Special attention:

- "set value, then validate";
- "initialize defaults, then check ownership";
- "write config, then discover parent is invalid";
- external side effects inside a DB transaction;
- state changes before permission checks.

---

# Pass 30 — Exact behavior / style-convention scan

Before reporting style or convention issues, inspect repository precedent quantitatively enough to avoid arbitrary preferences.

Check newly introduced code against nearby dominant conventions for:

- braces around `if`;
- naming of booleans and protocol fields;
- aliasing generated types;
- error mapping;
- constants vs literals;
- batching;
- helper placement.

Only report style when:

- a formatter/linter does not already settle it; and
- the new code materially increases inconsistency or obscures correctness.

Prefer grouping many identical low-level occurrences into one finding instead of flooding the review.

---

# Pass 31 — Reviewer-style final sweep

Before `CLEAN`, perform a final search-driven sweep independent of the implementation author's structure.

Search for:

- each new constant's literal value;
- each new Mongo/SQL predicate shape;
- each new helper's semantic equivalent;
- each new system identity name;
- each moved method's old tests;
- every call to the changed repository/service method;
- every creator/importer of the affected entity;
- every reader of newly persisted fields;
- every error string that signals supposedly impossible/corrupt state.

The goal is to catch facts that are invisible when reading files in the order they were changed.

Do not stop at the diff if a repository-wide search can falsify an assumption cheaply.

---

# Finding classification

Before recommending a fix, classify every concern.

Use one of:

- `BUG`
- `REGRESSION`
- `TEST_GAP`
- `CONTRACT_INCONSISTENCY`
- `PERFORMANCE_DEFECT`
- `MAINTAINABILITY`
- `PRE_EXISTING`
- `OUT_OF_SCOPE`
- `SPECULATIVE`

A finding is not automatically an instruction to modify the current MR.

### Fix-now rule

Normally recommend fixing now when the issue is:

- introduced or materially exposed by the current change;
- a reachable correctness defect;
- a meaningful regression;
- data loss/corruption risk;
- security/access-control risk;
- a test that falsely claims to protect changed behavior;
- a configuration/migration defect required for safe deployment.

Normally do not force an in-scope fix when the issue is:

- unrelated pre-existing debt;
- speculative performance work with no demonstrated problem;
- cleanup far outside the semantic radius;
- a separate architectural redesign not needed for correctness.

Still report relevant pre-existing issues when the current change directly activates, relies on, or materially increases their impact.

---

# Severity

## BLOCKER

Use when merge is clearly unsafe.

Examples:

- data corruption or destructive behavior;
- severe access/security issue;
- migration that can break production data/schema;
- startup failure for normal valid production data;
- fundamentally incorrect implementation.

## HIGH

Likely production defect or substantial behavioral regression.

## MEDIUM

Real correctness risk, test gap, duplicated business rule, conflicting contract, or maintainability defect that should normally be addressed before merge.

## LOW

Minor but legitimate issue.

Do not flood the review with LOW findings.

---

# Evidence requirements

Every finding must explain:

- location;
- what is wrong;
- why it matters;
- how the state is reached;
- what evidence supports it;
- classification;
- severity;
- recommended direction;
- whether it should be fixed in this MR.

Prefer concrete evidence:

- call hierarchy;
- writer/reader mismatch;
- surviving mutation;
- exact missing assertion;
- repeated DB lookup;
- destructive shared-state cleanup;
- mismatched migration `down`;
- missing env declaration;
- contradictory production paths;
- lifecycle path missing required initialization;
- pre-write vs post-write ownership state;
- repository-wide duplicate helper/constant/filter evidence;
- deleted test with no equivalent replacement;
- source-of-truth mismatch across components;
- avoidable persistence invariant introduced by storage shape.

Avoid vague statements such as:

> This could possibly be cleaner.

---

# Anti-noise rules

Do not report:

- personal style preferences;
- arbitrary renaming;
- generic "could be cleaner";
- theoretical abstractions with no concrete benefit;
- speculative edge cases not reachable from the domain;
- formatting that automated tooling already guarantees unless the branch is actually unformatted;
- unrelated pre-existing issues;
- broad redesigns outside scope.

Do not manufacture findings to avoid `CLEAN`.

A strict review must remain truthful.

---

# Existing unrelated failures

Do not modify production code or tests just to make unrelated known failures green.

If unrelated local/dev tests are known to fail while CI or the target environment is healthy:

- mention only if they block verification;
- do not treat them as findings against the change;
- do not change unrelated behavior to satisfy them.

---

# Mandatory adversarial gate before CLEAN

Do not return `CLEAN` until all applicable checks are satisfied:

- [ ] Intended behavior reconstructed.
- [ ] Every production creator/importer/migration/bootstrap path for affected entities enumerated.
- [ ] Relevant callers/callees/usages inspected.
- [ ] Every writer -> storage -> reader contract traced for changed persisted values.
- [ ] Missing-record/default/legacy semantics explicitly checked.
- [ ] Nullable/optional/empty/whitespace semantics checked where relevant.
- [ ] Create/import/migration/legacy paths compared for semantic consistency.
- [ ] Ownership/authorization checks evaluated against state **before** the operation mutates it.
- [ ] Failure injected mentally after each significant side effect; no forbidden partial persistence remains.
- [ ] New guards/branches/fallbacks proven reachable or justified.
- [ ] No important dead/unreachable code remains.
- [ ] Repository-wide search performed for equivalent helpers/constants/filters/registries introduced by the MR.
- [ ] No duplicated business knowledge with divergence risk remains.
- [ ] Security-relevant registries/source-of-truth definitions were checked for drift.
- [ ] New persistence structures challenged against the existing aggregate/default-by-absence model where applicable.
- [ ] Service/class responsibility remains coherent after moved logic.
- [ ] DB/external calls checked for repeated work and unnecessary rereads.
- [ ] Repeated writes checked for write amplification.
- [ ] Transaction/lock semantics inspected where relevant.
- [ ] Removed/moved tests mapped to replacement behavioral coverage.
- [ ] Important tests challenged with realistic mutations, including type/ownership discriminators and missing dependent records.
- [ ] Assertions prove the behavior named by the tests rather than mock topology.
- [ ] Mocks do not hide the implementation under test.
- [ ] Shared e2e/integration state is restored and isolated.
- [ ] Migrations were checked in both directions and against skipped-version upgrade paths where relevant.
- [ ] Migration tests actually exercise behavior that can fail.
- [ ] New config values have a complete deployment lifecycle.
- [ ] Removed features have no unsafe dead config/secrets/docs.
- [ ] Degraded behavior is diagnosable where necessary.
- [ ] User-visible formatting is consistent across equivalent paths.
- [ ] Final reviewer-style repository search sweep completed.
- [ ] Findings were classified as fix-now vs pre-existing/out-of-scope/speculative.
- [ ] No meaningful issue from the adversarial pass remains.

If an applicable item cannot be verified because context is unavailable, do not claim full `CLEAN`.

State the limitation instead.

---

# Output format

Return findings first, ordered by severity.

For every finding use:

## [SEVERITY] Short finding title

**Classification:** `BUG | REGRESSION | TEST_GAP | CONTRACT_INCONSISTENCY | PERFORMANCE_DEFECT | MAINTAINABILITY | PRE_EXISTING | OUT_OF_SCOPE | SPECULATIVE`

**Location:** file / method / test

**Problem:**  
Concrete description.

**Why it matters:**  
Production, behavioral, test-quality, operational, or maintainability consequence.

**Reachability / evidence:**  
Actual execution path, contract mismatch, surviving mutation, query pattern, shared-state effect, or other concrete proof.

**Recommended direction:**  
Describe the preferred correction without unnecessarily rewriting the implementation.

**MR scope:**  
`Fix in this MR` or `Separate task / no mandatory change`, with one short reason.

After findings include:

## Review coverage

Briefly state which relevant areas were actually inspected:

- behavior and entity lifecycle;
- production execution paths, including imports/scripts/migrations;
- contracts and all persistence writers/readers;
- ownership/authorization transitions and failure atomicity;
- reachability;
- repository-wide duplication/source-of-truth search;
- tests, deleted coverage, and mutations;
- mocks;
- persistence model and I/O;
- migrations/config and upgrade paths when applicable;
- operability;
- architecture and service responsibility.

Do not dump internal chain-of-thought.

If no findings remain:

# CLEAN

No actionable findings were identified after the full senior review pipeline.

Then briefly state review coverage.

Do not output `CLEAN` if relevant portions could not be meaningfully inspected.

---

# Review behavior

Be skeptical but evidence-driven.

Prefer one strong finding over five speculative ones.

Do not optimize for politeness or for finding something at all costs.

Optimize for catching defects before a human reviewer has to point them out.

Passing tests, compilation, lint, and previous reviews are evidence, not proof.
