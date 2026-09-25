---
name: senior-review
description: Strict senior-level production pre-MR review. Reconstructs behavior and lifecycles, traces contracts and persistence, attacks authorization and failure atomicity, proves reachability, searches repository-wide for duplicated business knowledge, challenges tests with realistic mutations, audits event/delivery/idempotency semantics, hot-path cost, migrations, configuration, operability, and cleanup. Findings must be evidence-based and scoped to the MR.
---
# Senior Review
## Purpose
Perform a strict production-grade review intended to discover defects an experienced human reviewer would find after reading beyond the diff.

The goal is not to confirm that the implementation looks reasonable. The goal is to falsify it.

You must:
- reconstruct intended behavior and preserved behavior;
- identify negative guarantees: what must remain impossible or forbidden;
- trace real production execution paths, not only changed methods;
- enumerate every meaningful creator/writer/reader of changed state;
- verify contracts across service, persistence, event, and client boundaries;
- prove new branches, fallbacks, and defensive checks are reachable;
- attack authorization, ownership, idempotency, and failure atomicity;
- search repository-wide before accepting new helpers, constants, filters, registries, or abstractions;
- challenge tests with realistic mutations and production-shaped fixtures;
- inspect database, serialization, fan-out, and repeated-execution cost;
- verify migrations, configuration, observability, and cleanup;
- distinguish real defects from pre-existing debt, speculation, or unrelated redesign.

Assume subtle defects may exist even when:
- code compiles;
- lint passes;
- tests pass;
- the diff is small;
- the implementation looks clean;
- another review returned `CLEAN`.

Do not invent issues. Every finding needs concrete evidence from code, contracts, tests, persistence behavior, execution paths, measurements, or a realistic production scenario.
---
# Core review model
Use this order unless a repository-specific reason requires a different one:
1. Behavior, lifecycle, and negative guarantees
2. Semantic neighborhood and repository-wide context
3. Contracts, persistence, and normalization
4. Symmetric production paths and reachability
5. Correctness, async flow, and failure atomicity
6. Authorization, ownership, and reserved identities
7. Business knowledge and source of truth
8. Redundancy, cleanup, and refactor residue
9. Database, I/O, hot-path, and repeated-execution cost
10. Transactions and persistence model
11. Tests and mutations
12. Migrations, configuration, and operability
13. Event transport, idempotency, delivery, and protocol boundaries
14. Architecture and framework-specific responsibility
15. Final adversarial sweep

Do not skip later areas because earlier ones look clean.
---
# Pass 1 — Behavior, lifecycle, and negative guarantees
Before judging implementation, determine:
- what behavior is introduced, removed, or changed;
- what existing behavior must remain unchanged;
- what must still be impossible or forbidden;
- which actors and execution paths are affected;
- which side effects can occur;
- which invariants must remain true;
- what failures are acceptable, visible, retryable, or fatal;
- every way the affected entity/state can come into existence.

For each important entity enumerate applicable lifecycle entrypoints:
`public API -> internal API/gRPC -> admin/CLI -> import/script -> migration/backfill -> seed/bootstrap -> sync/reconciliation -> restore/replay -> legacy data -> direct repository writer`

Do not assume the changed method is the only creator or writer.

Creation paths that are not user-facing still matter if an internal contract can create an unsafe state.

For each lifecycle path ask:
- Does it establish the same ownership and defaults?
- Does it create required dependent state?
- Is the entity externally visible before initialization completes?
- Can readers observe partial initialization?
- Can historical or restored records legitimately omit newly introduced state?

If correctness relies on an invariant such as “every bot has configuration X”, prove which component establishes it for every lifecycle path and when it becomes true.

### Negative guarantee preservation
For every removed field, guard, validation, registry, or branch ask:
> What bad state did this code make impossible?

Map:
`removed protection -> protected bad state -> new enforcement point -> test proving preservation`

If no replacement enforcement point exists, report a regression even if the old mechanism was intentionally removed.

### Case matrix
Use a small domain-relevant matrix when helpful:
- normal input;
- missing/empty/null input;
- stale or legacy persisted data;
- duplicate/repeated execution;
- invalid/partial external payload;
- dependency failure;
- multi-user or multi-consumer execution;
- boundary date/time or protocol-version case.

Do not mechanically enumerate irrelevant edge cases.

Tests and production code are independent evidence. Neither is automatically the specification.
---
# Pass 2 — Semantic neighborhood and repository-wide context
Do not review a changed line in isolation.

Inspect relevant:
- callers and callees;
- usages/call hierarchy;
- sibling methods and equivalent flows;
- interfaces and generated contracts;
- DTOs, schemas, validators, parsers, and normalizers;
- repositories, models, workers, cron jobs, hooks, and commands;
- guards/interceptors;
- migrations and configuration;
- docs and tests;
- similar implementations elsewhere.

Expand only as far as necessary to understand real behavior.

Prefer usages/call hierarchy over text search when available.

For every new helper, constant, predicate, registry, adapter, transaction wrapper, stream bridge, generic utility, or magic value search repository-wide for:
- same symbol;
- same literal;
- equivalent filter/query shape;
- equivalent call pattern;
- sibling implementation with another name.

Different names do not prove different concepts.

Ask:
> If this value or rule is wrong here, what is the furthest downstream consequence?
---
# Pass 3 — Contract, persistence, and normalization flow
For every important changed value trace:
`external input -> parser/validator -> canonical representation -> persistence/event -> readers -> comparisons -> output/side effects`

Pay special attention to:
- IDs and foreign IDs;
- usernames/system identifiers;
- dates/timezones;
- email/URL values;
- enum-like strings;
- nullable/optional fields;
- arrays and nested transport structures;
- secrets/tokens;
- snapshots and legacy persisted values.

Determine:
- where normalization belongs;
- what the canonical internal representation is;
- whether `null`, `undefined`, `''`, and whitespace differ;
- whether all writers produce the same form;
- whether all readers expect the same form;
- what a missing persisted record means;
- whether absence is a valid default, legacy state, or corruption;
- whether a migration/backfill is required for correctness or only compensates for storage design.

### Normalization ownership
Normalization should normally have one owner.

Flag semantic duplication when:
- one writer trims and another does not;
- one path maps enum/string values differently;
- a boundary canonicalizes but downstream paths reinterpret again;
- fresh data is canonical but historical data is not accounted for;
- two readers disagree about nullable or optional semantics.

Do not report harmless repeated formatting unless it creates conflicting semantics or hidden assumptions.

### Mandatory writer inventory
For every changed persisted concept build an internal table:
`writer/path | preconditions | value written | transaction | can record be absent? | affected readers`

Include imports, scripts, migrations, bootstrap, direct repository writers, and legacy data.

A persistence invariant is not proven until every writer/creator path is checked.

### Read-time validation reachability
For each read-time validator ask:
1. Who can write the field?
2. Do all real writers already validate/normalize it?
3. Can legacy, migration, manual, admin, or older-version paths bypass them?
4. Can storage become malformed independently?
5. What reachable state does this validator reject?
6. What changes if the validator is deleted?

If the rejected state cannot be produced by any real writer, classify the branch as redundant/unreachable rather than “defensive correctness”.

Do not remove read validation when the DB is shared, manually edited, written by old versions, or has independent writers without proving writer exclusivity.
---
# Pass 4 — Semantic symmetry and equivalent production paths
Compare neighboring paths that conceptually implement the same rule:
- create vs update/complete-existing;
- API vs worker/cron;
- manual vs synchronization;
- user-owned vs system/bootstrap;
- create vs import/migration/restore;
- fresh vs legacy entity;
- admin vs ordinary user;
- snapshot writer vs cleanup reader;
- two consumers of the same external payload;
- two transports exposing the same capability.

Ask:
- Do they define the concept the same way?
- Can one path undo another?
- Does one clear stale state while another preserves it?
- Do they use the same identity, timezone, normalization, fallback, and authorization rules?
- Does one grant a capability another later revokes?

Contradictions between valid production paths are usually more important than local style issues.
---
# Pass 5 — Reachability and defensive-code proof
For every meaningful new branch, fallback, guard, validator, or defensive check ask:
> Can this state actually occur through a production path?

Inspect especially:
- `if` branches and `??` fallbacks;
- optional arguments;
- impossible combinations of derived fields;
- cycle protection on structures guaranteed acyclic by parser/schema;
- checks duplicated by an earlier invariant;
- branches constructible only by tests.

A unit test does not prove a state is realistic.

If production guarantees `B = f(A)`, a test that sets `A` and `B` independently may create an impossible state.

When a branch only protects an impossible state:
- remove it; or
- document the invariant; or
- prove the production scenario that requires it.

Do not keep unreachable code merely because it is testable.
---
# Pass 6 — Correctness, async flow, and failure atomicity
Check for:
- wrong/inverted conditions;
- missing branches;
- stale state;
- accidental broadening/narrowing of access;
- incorrect fallback or state transition;
- unintended mutation;
- swallowed/misclassified errors;
- incorrect error propagation;
- races;
- missing `await`;
- `Promise` used as boolean;
- async work started but not observed;
- work outside intended transaction/try-catch;
- read-check-write races;
- state mutated before authorization completes;
- durable partial state after failure.

For each mutating request write side effects in order:
`read -> write A -> external call -> check -> write B -> response`

Inject a failure after every significant step.

Verify:
- unauthorized/conflicting operations leave no forbidden mutation;
- a later `INTERNAL`, `ALREADY_EXISTS`, validation, or access error does not leave earlier writes behind;
- related writes are transactional or safely idempotent/recoverable;
- one logical operation is failure-atomic when its caller expects it to be.

Ask:
> If the method throws on the next line, what has already been persisted or externally emitted?

Pay special attention to:
- “set value, then validate”;
- “initialize defaults, then check ownership”;
- “write config, then discover parent is invalid”;
- external side effects inside transactions;
- state changes before permission checks.
---
# Pass 7 — Authorization, ownership, and reserved identities
When role/capability/ownership logic changes, trace every consumer and transition.

Inspect:
- guards/interceptors;
- admin/internal paths;
- background synchronization;
- impersonation;
- tenant/department/subtree scope;
- cleanup/reconciliation;
- existing-entity/claim/attach flows.

For create-or-complete, claim, attach, import, activation, and existing-entity paths trace:
`state before call -> first write -> authorization/ownership check -> later writes -> externally visible result`

Verify:
- authorization uses pre-existing trusted state, not state just initialized from caller input;
- unowned/legacy/imported entities cannot be claimed merely by supplying an owner;
- insert-only defaults do not turn attacker input into authoritative ownership;
- forbidden operations leave no durable mutation;
- secrets/webhooks/permissions are not changed before identity/ownership is established.

A check performed after the operation establishes the condition being checked is not a valid security check.

### Reserved namespace
When trust depends on a well-known username, slug, route, tenant key, service account, or system bot name:
- locate where reservation is enforced;
- verify user-controlled creation cannot preempt it;
- verify removal of `is_system`, `reserved`, or similar state did not remove the only protection;
- distinguish “who marks it system” from “who prevents ordinary users from occupying the namespace”;
- inspect bootstrap/ensure logic that trusts an already-existing record;
- ensure ordinary fixtures do not accidentally use reserved identities.

Treat a trusted namespace as an authorization boundary.
---
# Pass 8 — Duplicated business knowledge and source of truth
Differentiate duplicated syntax from duplicated business rules.

Prioritize duplicated knowledge such as:
- authorization predicates;
- timezone/normalization rules;
- relation/policy logic;
- event formatting semantics;
- composite/idempotency keys;
- state transitions;
- filtering rules;
- reserved/system identity lists;
- capability registries;
- duplicate-key/error codes;
- retry/batch constants.

Ask:
> If one copy changes next month, can the other silently remain stale?

For registries, allowlists, deny-lists, enum mirrors, system identities, or protocol constants:
1. locate every definition;
2. identify the authoritative owner;
3. verify copied entries are actually provisioned/created;
4. verify consumers agree on semantics.

Security-relevant duplicated truth is a correctness risk, not merely DRY cleanup.

Search both by symbol and literal/structural shape.

If copies already diverged, report the concrete divergence.

Do not demand abstractions for every repeated line.
---
# Pass 9 — Redundancy, deletion, and refactor residue
Inspect:
- dead code and unused public methods;
- duplicate guards/validation/conditions;
- repeated transformations/parsing;
- no-op fallbacks and unnecessary object reconstruction;
- repeated queries;
- stale comments/JSDoc;
- obsolete imports, aliases, config, docs, examples, secrets, and dependencies;
- test names describing deleted behavior;
- fixtures carrying special semantics accidentally.

For every meaningful new line ask:
> Why does this line need to exist?

Mentally remove:
- branch;
- helper;
- fallback;
- wrapper;
- validation layer;
- transformation;
- field from an update.

If intended behavior remains unchanged, investigate whether it is unnecessary.

### Feature deletion cleanup
When functionality is removed, inspect its whole footprint:
`imports -> providers -> dependencies -> types -> env/config -> credentials/secrets -> deployment -> docs -> tests -> migrations -> examples/comments`

Removing code while leaving dead secrets/config/docs is incomplete cleanup.

### Comment attachment
After deleting/moving the line a comment described, verify the comment still describes the immediately following construct.

### Semantic fixtures
Test data is not neutral when usernames, IDs, roles, system names, sentinel values, or error codes have production meaning.
---
# Pass 10 — Database, I/O, and hot-path cost
Inspect production paths for repeated work:
- N+1 repository calls;
- repeated loads of the same entity;
- DB/external calls inside loops;
- same query with identical arguments;
- broad reads followed by narrow in-memory filtering;
- expensive work before cheap guard/dedup checks;
- deep cloning or serialization repeated per recipient;
- capability parsing repeated per frame/recipient;
- message × recipient work that can be message-only or connection-only.

Prefer, when semantics allow:
`cheap rejection/dedup -> stable per-operation lookup -> DB/external I/O -> expensive transformation`

### Pass-local memoization
If data must be fresh between requests but is invariant across one execution, consider memoizing by stable key only for the lifetime of that operation.

Do not recommend long-lived caching unless freshness is understood.

### Fan-out serialization
When an event is serialized once per recipient, multiply cost by realistic recipient cardinality.

Check for:
- fields needed only by one recipient class but serialized for everyone;
- repeated deep clones;
- repeated DB lookups inside message × recipient loops;
- repeated encoding of identical shared fragments;
- transformations that turn O(messages) into O(messages × recipients).

When measurable, estimate:
`payload growth × recipients`

and

`serialization time growth × recipients`

A material cost for data unnecessary to most recipients is a performance defect, not generic optimization advice.
---
# Pass 11 — Write amplification and repeated execution
For each write in cron, sync, worker, reconciliation, polling, retry, or frequently called path ask:
> How many writes, how often, across how many entities?

Estimate:
`writes per execution × executions per period × entities`

Check for:
- blanket updates;
- unchanged values rewritten every run;
- per-row updates replacing bulk operation;
- large transactions around long loops;
- unnecessary write locks;
- writes before change detection.

Handle `NULL` comparisons correctly where applicable.

A write harmless once may be unacceptable every minute across thousands of records.
---
# Pass 12 — Transaction and locking semantics
Inspect:
- transaction boundaries;
- lock duration/type;
- hidden `FOR UPDATE` semantics;
- external calls inside transactions;
- loops inside one transaction;
- nested/optional transaction behavior;
- repository names that hide locking.

A normal-looking repository method should not silently become a locking read merely because a transaction was passed unless that convention is explicit and established.

Performance cost alone is not automatically merge-blocking; classify by evidence.
---
# Pass 13 — Aggregate and persistence-model challenge
When a branch introduces a collection/table/config store for data belonging to an existing entity ask:
- Does the parent already provide natural identity/lifecycle?
- Is the data one-to-one with the parent?
- Is it normally read together with the parent?
- Does the new store require eager default rows/documents?
- Does absence need special errors or migrations?
- Does the split create transactions solely to keep records consistent?
- Does it create multiple encodings of “empty”?
- Does it add mostly pass-through repository/service layers?
- Does the codebase already store similar fields on the aggregate?

Compare:
`fields/subdocument on existing aggregate`
vs
`separate EAV/config collection/table`

Do not prescribe one universally. Report concrete cost: extra invariants, migrations, transactions, failure modes, I/O, or plumbing without an independent lifecycle/query requirement.

### Default by absence
For optional configuration, consider whether missing data can safely mean the domain default instead of materializing a record.
---
# Pass 14 — Test intent and requirement validity
For each added/changed test determine:
1. What behavior does the name claim?
2. What setup actually reaches?
3. What do assertions actually prove?
4. Could an earlier unrelated call satisfy the assertion?
5. Is it checking observable behavior or implementation topology?
6. Does it execute the real function whose correctness matters?
7. Does the fixture represent a reachable production state?

A passing test is not evidence the test is useful.

### Deleted or moved coverage
When logic moves or tests are deleted:
1. identify removed behavioral claims;
2. map each claim to replacement coverage;
3. verify the replacement fails if the corresponding production rule is removed;
4. do not accept “covered elsewhere” without locating the exact assertion.

Check removed coverage for:
- not-found behavior;
- type discrimination;
- ownership conflicts;
- reserved-name rejection;
- already-existing/disabled behavior;
- invalid legacy state;
- error mapping.

For every removed guard, identify the test that would fail if it disappeared.

### Tests that codify accidents
Be suspicious of tests that preserve branch history rather than requirements, for example tests that:
- assert absence/presence of methods unrelated to supported behavior;
- preserve a temporary format that never shipped;
- assert duplicate suppression without proving intended idempotency scope;
- verify a hidden/filter flag after the producer stopped emitting it;
- test only an RPC filter while live streams bypass that path;
- lock in two names for one protocol field.

Ask:
> What user/system requirement would fail if this test were deleted?

If the answer is only “it documents how the branch evolved”, delete or rewrite it around supported behavior.
---
# Pass 15 — Mutation, assertion strength, and mock integrity
For each significant behavior, choose the smallest realistic mutation that should make a test fail.

Useful mutations include:
- remove/negate a condition;
- return a superset or subset;
- remove an important `where`/update field;
- remove normalization;
- omit `await`;
- return a constant;
- skip persistence;
- call a dependency one extra time;
- run scenario twice;
- use two users/consumers;
- break the real implementation hidden behind a mock;
- remove type/ownership/authorization discriminator;
- make dependent config absent;
- route creation through import/bootstrap/migration;
- throw immediately after first persistence side effect;
- remove null/not-found guard;
- allow user-owned creation with reserved identifier;
- replace neutral fixture with reserved/system value;
- replace handwritten type with generated equivalent;
- remove read-time validation for canonical-only data.

If the test stays green under a realistic mutation related to its claimed behavior, report the surviving mutation and missing assertion/scenario.

### Assertion strength
Inspect:
- broad `arrayContaining`/`objectContaining`;
- `toHaveBeenCalledWith` matching any call;
- `toThrow()` without reason;
- snapshots hiding important extras;
- call-count omissions;
- test names promising more than assertions verify.

Use exact collections, counts, `NthCalledWith`, explicit negative assertions, or observable output when the behavior requires them.

### Mock integrity
Look for:
- mocking the exact pure function the scenario claims to test;
- reimplementing production logic in test helpers;
- stubs that always return the expected assertion value;
- impossible derived-field combinations;
- helper defaults hiding required arguments;
- mocks bypassing parser/guard/repository behavior relevant to the change.

Ask:
> If the real implementation breaks, can this test remain green because the test replaced it?
---
# Pass 16 — Shared integration/e2e state
Inspect lifecycle/isolation:
- `beforeAll` / `beforeEach` / `afterEach` / `afterAll`;
- global setup;
- shared DB/schema;
- shared Redis/queue state;
- migrations;
- destructive cleanup.

Treat as high risk until justified:
- `destroy({ where: {} })`;
- truncate/delete-all;
- `migration.down()`;
- dropping columns/tables;
- changing shared config/state.

Ask:
- Is schema/state restored?
- Does cleanup remove only records owned by the test?
- Can order change the result?
- Can Jest caching/timing hide the problem?
- Would this pass on a clean CI runner?

Tests must not leave shared state broken for later suites.
---
# Pass 17 — Migration and upgrade-path audit
Review migrations in both directions.

For `up` inspect:
- existing-row compatibility;
- defaults/nullability;
- indexes/constraints;
- data transformation;
- locking/large-table impact when relevant.

For `down`:
- compare against the actual schema before `up`;
- restore original nullability/default/index/constraint behavior;
- do not invent a different historical schema.

If tests run `down()` on a shared DB, verify they restore schema afterwards or isolate it.

### Upgrade path
Do not reason only about `previous version -> target version`.

Consider supported jumps from older releases and legacy persisted data.

A valid historical entity must not become corrupt because it missed application-level initialization introduced only in an intermediate release.

If a backfill exists only because optional configuration was split away from an aggregate, compare the resulting complexity with default-by-absence.

Migration tests must observe behavior the migration could actually violate; tautological mock assertions provide no protection.
---
# Pass 18 — Configuration and operability
For each new/changed config inspect:
- code reader (`get` vs `getOrThrow`);
- defaults;
- env validation;
- example env;
- Docker/Compose/deployment manifests;
- tests;
- docs.

Classify as:
- required;
- optional with meaningful default;
- optional nullable.

If correctness, access control, destructive behavior, integration identity, or important user behavior depends on the value, prefer fail-fast startup over a fake placeholder default.

### Observability
A safe fallback can still be operationally dangerous if nobody can see it.

Inspect:
- warnings on degraded behavior;
- error context and identifiers;
- swallowed dependency failures;
- retries misclassified as another failure;
- logs that only contain generic messages.

Require diagnosability when behavior materially degrades, data is skipped, access changes, or support would otherwise be blind.
---
# Pass 19 — User-visible and serialized representation consistency
When the change affects text, dates, templates, payload names, or client-visible serialization inspect the final rendered/encoded result.

Check:
- all-day vs timed events;
- timezone/date formatting;
- fallback wording;
- punctuation/technical labels;
- equivalent messages from different paths;
- field naming consistency;
- accidental dual snake_case/camelCase exposure;
- template conditions broadening visibility.

Do not assume reuse of a generic formatter is correct if surrounding semantics differ.

Evaluate the actual user/client-visible output.
---
# Pass 20 — Generated contracts and object merge semantics
### Generated contract duplication
When protobuf/OpenAPI/GraphQL/codegen types exist, search for handwritten local types mirroring them.

Ask:
- Is it field-for-field identical?
- Does it add domain semantics, narrowing, branding, validation, or internal-only state?
- Is decoupling intentional?
- Or is it only a drift-prone duplicate?

Flag duplicates only when the generated contract is already canonical and the local type adds no independent meaning.

### Object spread/merge
Treat spreads as executable merge logic.

Inspect:
- precedence;
- raw values overwriting normalized values;
- `undefined` erasing valid values;
- stale fields being preserved;
- defaults overriding explicit values;
- confusing conditional spreads.

Do not flag harmless spreads for style alone.
---
# Pass 21 — Event transport, visibility, and routing
Treat broker/DDP/WebSocket/Redis/gRPC events as behavioral contracts.

For each event field or synthetic event trace:
`producer -> serializer -> broker/stream -> fan-out -> gateway/adapter -> client-visible frame`

Verify:
- intended audience: all room members, one user, bots only, internal consumers only;
- internal routing metadata is removed from ordinary recipients;
- every transport applies the required filtering/renaming;
- live streams do not bypass filters used only by request/response methods;
- the same logical field does not leak under two names;
- capability-gated representation is consistent across boundaries.

### Internal-record leakage
If transport/control actions are stored in a general-purpose message/event collection, enumerate every reader:
- delta sync;
- room history;
- unread/read-state calculations;
- notifications;
- search/indexing;
- exports/audit;
- live room streams;
- generic created/updated handlers.

A hidden/internal flag protects nothing unless every externally reachable reader honors it.

If the public contract cannot represent the hidden state, leakage risk is higher.

Prefer a distinct event/type/store when an action is not semantically a chat message and the message model requires many exclusion rules.
---
# Pass 22 — Idempotency, deduplication, and delivery semantics
For commands, callbacks, button presses, jobs, webhooks, queue messages, and retries reconstruct the exact idempotency key.

Ask:
- Which dimensions are included: actor, tenant, room, source, action, attempt, time window?
- Can two legitimate users collide?
- Can one user legitimately execute twice?
- Does dedup suppress retries only, or valid independent actions?
- What durable state is written before actual delivery?
- What happens after producer/consumer restart?

### Delivery-state semantics
Do not equate successful publisher invocation with delivery unless the API contract guarantees it.

Map states precisely, for example:
`created -> accepted -> published -> consumed/acknowledged -> processed`

If persistence says `delivered` after only `publish()`/`publishAndWait()` without receiver acknowledgement, flag the semantic mismatch.

### Minimum multi-actor scenarios
Test:
1. same actor, same action repeated;
2. different actor, same action;
3. same actor, distinct legitimate action;
4. retry while consumer unavailable;
5. retry after restart.

A duplicate-key test proving only “no second event” is insufficient when actors/actions must remain distinguishable.
---
# Pass 23 — Compatibility and capability/protocol adaptation
### Real compatibility vs branch history
For each legacy/compatibility branch determine:
- which released/tagged/deployed version wrote the old format;
- whether it ever existed in mainline/release tags/production data;
- whether migrations/imports/external clients can still produce it;
- whether tests protect real compatibility or only feature-branch history.

If no shipped producer existed, treat the branch as refactor residue/dead complexity, not backward compatibility.

For retained historical fields ask:
- are they read after write?
- do they retain secrets/tokens unnecessarily?
- are stale values bounded/expired?
- would an operation ID or smaller marker preserve the real invariant?

Do not accumulate revoked/sensitive values merely for idempotency if only a marker is required.

### Capability/protocol adaptation
Identify the canonical capability parser/accessor and inspect every boundary for bypasses.

Verify:
- HTTP headers/query, DDP params, WebSocket handshake, gRPC metadata, etc. normalize consistently;
- code uses canonical accessors rather than ad-hoc parsing;
- stable capabilities are not recomputed per frame unnecessarily;
- capability transforms apply to both RPC responses and live streams when required;
- removing a method does not leave filters/tests that only served that method.

If two parsers exist, compare syntax, defaults, normalization, and error behavior.
---
# Pass 24 — Architecture, responsibility boundaries, and framework specifics
When logic moves between service/class/module boundaries, review the move as a unit.

Map operations for the same aggregate/capability:
`create | get | update | disable/delete | assemble/map | validate | authorize | deliver`

Ask:
- Did only half of one responsibility move?
- Does a controller/gateway now coordinate multiple services for one domain action?
- Did moving logic lose an existing mapper and introduce rereads?
- Are validation/identity/authorization rules duplicated across layers?
- Did a thin adapter become a second domain service?

Concrete boundary drift includes:
- direct model/repository access beside an existing service abstraction;
- authorization duplicated beside a guard/interceptor;
- different error codes for the same rule;
- manual validation beside established DTO validation;
- duplicated delivery/state-transition branches;
- idempotency/persistence logic in transport code.

The problem is not line count. Prove duplicated ownership, divergence risk, or extra I/O.

### NestJS / TypeScript checks
When applicable inspect:
- DI/module exports/provider scope;
- lifecycle hooks and startup failure radius;
- guards/interceptors and exception mapping;
- missing `await`;
- nullable/optional runtime mismatch;
- unsafe assertions;
- DTO/runtime validation mismatch;
- transaction boundaries;
- worker idempotency/cron overlap;
- repeated repository calls;
- domain capability split across old/new services.

Use the dedicated `nestjs-review` skill for deeper framework-specific coverage when relevant.
---
# Pass 25 — Style-convention sanity and final adversarial sweep
Do not turn senior review into arbitrary style review.

Before reporting convention issues, inspect nearby repository precedent for:
- boolean/protocol naming;
- generated-type usage;
- error mapping;
- constants vs literals;
- batching/helper placement;
- braces/format only when tooling does not settle it.

Report style only when inconsistency materially obscures correctness or increases drift risk.

### Final search-driven sweep
Before `CLEAN`, independently search for:
- new constants' literal values;
- new DB predicate shapes;
- semantic equivalents of new helpers;
- system/reserved identity names;
- old tests for moved methods;
- all callers of changed service/repository methods;
- all creators/importers of affected entities;
- all readers of new persisted fields;
- supposedly impossible/corrupt-state error strings;
- deleted guards and replacement enforcement points;
- handwritten types matching generated contracts;
- comments beside moved/deleted code;
- fixtures with special production semantics.

The final sweep is a verification step, not a second full review. Do not repeat analyses already completed unless the search reveals contradictory evidence.
---
# Finding classification
Classify each concern as one of:
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

## Fix-now rule
Normally recommend fixing in the current MR when the issue is:
- introduced or materially exposed by the change;
- a reachable correctness defect;
- a meaningful regression;
- data loss/corruption risk;
- security/access-control risk;
- a test falsely claiming to protect changed behavior;
- a migration/configuration defect required for safe deployment.

Normally do not force an in-scope fix when it is:
- unrelated pre-existing debt;
- speculative performance work without evidence;
- cleanup outside the semantic radius;
- a broad redesign unnecessary for correctness.

Still report pre-existing issues when the change directly activates, relies on, or materially increases their impact.
---
# Severity
## BLOCKER
Use when merge is clearly unsafe, for example:
- data corruption/destructive behavior;
- severe security/access issue;
- migration breaking production schema/data;
- startup failure for valid production data;
- fundamentally incorrect implementation.

## HIGH
Likely production defect or substantial behavioral regression.

## MEDIUM
Real correctness risk, test gap, duplicated business rule, contract conflict, or maintainability defect that should normally be addressed before merge.

## LOW
Minor but legitimate issue. Do not flood the review with LOW findings.
---
# Evidence requirements
Every finding must explain:
- location;
- what is wrong;
- why it matters;
- how the state is reached;
- concrete evidence;
- classification;
- severity;
- recommended direction;
- MR scope.

Prefer evidence such as:
- call hierarchy;
- writer/reader mismatch;
- surviving mutation;
- missing assertion;
- repeated DB lookup;
- shared-state cleanup;
- mismatched migration `down`;
- missing required config;
- contradictory production paths;
- lifecycle path missing initialization;
- post-write ownership check;
- repository-wide duplicate rule/constant/filter;
- deleted test with no replacement;
- source-of-truth mismatch;
- reserved namespace preemption;
- hidden/control event leaking through generic readers;
- idempotency collision between actors/actions;
- `published` mislabeled as `delivered`;
- per-recipient serialization/clone amplification;
- branch-only compatibility with no shipped producer;
- local type identical to generated contract;
- unreachable read-time validation;
- avoidable persistence invariant introduced by storage shape.

Avoid vague statements such as:
> This could possibly be cleaner.
---
# Anti-noise rules
Do not report:
- personal style preferences;
- arbitrary renaming;
- generic “could be cleaner”;
- theoretical abstractions with no concrete benefit;
- speculative unreachable edge cases;
- formatting already guaranteed by tooling unless actually broken;
- unrelated pre-existing issues;
- broad redesigns outside scope.

Do not manufacture findings to avoid `CLEAN`.

A strict review must remain truthful.
---
# Existing unrelated failures
Do not modify production code or tests merely to make unrelated known failures green.

If unrelated local/dev tests fail while the target CI/environment is healthy:
- mention them only if they block verification;
- do not classify them against the change;
- do not alter unrelated behavior to satisfy them.
---
# Mandatory adversarial gate before CLEAN
Do not return `CLEAN` until applicable groups are verified:
- [ ] Intended behavior, preserved behavior, negative guarantees, and all production lifecycle entrypoints reconstructed.
- [ ] Relevant callers/callees/usages plus every writer -> storage/event -> reader contract traced.
- [ ] Missing/default/legacy and nullable/optional/empty semantics checked.
- [ ] Equivalent production paths compared for semantic symmetry.
- [ ] New guards/fallbacks/read validators proven reachable or identified as redundant.
- [ ] Failure injected after significant side effects; no forbidden partial persistence remains.
- [ ] Authorization uses trusted pre-mutation state; ownership and reserved namespaces cannot be preempted.
- [ ] Removed protections mapped to replacement enforcement and preserving tests.
- [ ] Repository-wide duplication/source-of-truth search completed for new business rules, constants, filters, registries, and helpers.
- [ ] Dead/refactor residue, stale comments/config/docs/tests, and semantic fixtures checked.
- [ ] DB/I/O, hot-path fan-out, repeated serialization, repeated queries, and write amplification evaluated where relevant.
- [ ] Transaction/lock semantics and persistence-model complexity inspected.
- [ ] Removed/moved tests mapped claim-by-claim; important behaviors challenged with realistic mutations and strong assertions.
- [ ] Mocks do not replace the behavior under test; shared integration state is restored/isolation-safe.
- [ ] Migrations checked up/down and across supported upgrade paths; configuration lifecycle and diagnosability checked.
- [ ] Generated/local types, serialized field names, object merges, and client-visible representations checked.
- [ ] Event visibility/routing checked across history/sync/read-state/streams and all relevant transports.
- [ ] Idempotency scope includes correct actor/action dimensions; delivery-state labels match actual transport guarantees.
- [ ] Compatibility branches correspond to real shipped/historical producers, not only feature-branch history.
- [ ] Capability parsing/adaptation is consistent across request/response and live-stream boundaries.
- [ ] Service/controller/gateway responsibilities remain coherent; no duplicated domain rule or avoidable reread was introduced.
- [ ] Final search-driven sweep completed and findings classified as fix-now vs pre-existing/out-of-scope/speculative.

If an applicable group cannot be verified because context is unavailable, do not claim full `CLEAN`. State the limitation.
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
Actual execution path, contract mismatch, surviving mutation, query pattern, event leak, idempotency collision, shared-state effect, or other concrete proof.

**Recommended direction:**  
Describe the preferred correction without unnecessarily rewriting the implementation.

**MR scope:**  
`Fix in this MR` or `Separate task / no mandatory change`, with one short reason.

After findings include:

## Review coverage
Briefly state which relevant areas were actually inspected, grouped rather than repeating every pass:
- behavior/lifecycle/contracts;
- authorization/failure atomicity;
- repository-wide consistency/reachability;
- tests/mutations/mocks;
- persistence/I/O/migrations/config;
- events/idempotency/protocol boundaries;
- architecture/operability.

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
