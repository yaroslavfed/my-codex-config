---
name: go-review
description: >
  Go-specific code review focused on correctness, idiomatic Go, concurrency safety,
  API/package design, maintainability, performance, tests, and Go conventions.
  Use for reviewing Go diffs, packages, pull requests, implementations, and refactors.
---

# Go Review

Perform a strict Go-specific code review.

The goal is not to maximize the number of comments. The goal is to find real defects,
regressions, non-idiomatic Go, unsafe concurrency, misleading APIs, unnecessary complexity,
and measurable or strongly justified performance problems.

Do not modify code unless the user explicitly asks for fixes.

Write the review, findings, explanations, conclusions, and recommendations in Russian.
Keep Go identifiers, commands, diagnostics, and technical terms in their original form where useful.

## Core principles

Review in this order:

1. correctness and behavioral regressions;
2. concurrency, cancellation, ownership, and resource lifetime;
3. error handling and API contracts;
4. idiomatic Go and package/interface design;
5. maintainability and unnecessary complexity;
6. tests and observability of failure modes;
7. performance and allocations;
8. style and mechanical conventions.

Do not elevate a style preference above correctness.

Do not demand abstractions, patterns, interfaces, helpers, layers, generics, goroutines,
channels, or dependency injection unless they solve a concrete problem in this code.

Prefer simple explicit Go over framework-style or Java/C#/TypeScript-style architecture.

A review finding must be actionable and supported by code, a call path, a documented Go rule,
a test gap tied to a realistic failure mode, a tool diagnostic, or a reproducible scenario.

Do not report speculative issues merely because something "could theoretically happen".

If there are no meaningful findings, return `CLEAN`.

---

# 1. Establish review scope

Before judging code:

- determine the diff or requested files;
- inspect `go.mod` and note the declared Go version;
- inspect affected packages, not just individual files;
- inspect callers and callees of changed exported or behaviorally important functions;
- inspect interfaces implemented or consumed by changed types;
- inspect relevant tests;
- inspect configuration, generated code, protocol definitions, database access, HTTP/gRPC handlers,
  workers, queues, and external integrations when the changed code participates in them;
- distinguish generated code from handwritten code and do not review generated style as if handwritten;
- determine whether behavior changed intentionally or accidentally.

When reviewing a branch or PR, prefer the merge-base diff over only the current working tree.

Do not assume a symbol is unused or safe to change without checking usages.

---

# 2. Correctness

Look for actual semantic defects first.

Check:

- wrong conditions, inverted booleans, broken boundary checks;
- incorrect zero-value assumptions;
- nil dereferences;
- typed-nil interface bugs;
- accidental shadowing with `:=`;
- ignored return values or ignored errors that affect correctness;
- incorrect slice bounds or mutation;
- aliasing bugs involving slices, maps, pointers, or reused buffers;
- unintended mutation through shared references;
- incorrect map presence checks where zero value and absence differ;
- incorrect `append` assumptions about backing arrays;
- stale references after append/reallocation;
- range-variable address/capture issues where applicable to the module's Go version;
- incorrect closure capture;
- overflow, truncation, signed/unsigned conversion, duration/unit mistakes;
- time-zone, monotonic-time, deadline, timer, and ticker misuse;
- incorrect parsing/formatting assumptions;
- partial success and partial failure handling;
- transaction boundaries and rollback behavior;
- duplicate processing and idempotency;
- retry behavior and retry amplification;
- ordering guarantees;
- invalid state transitions;
- inconsistent validation between entry points;
- behavior differences between nil and empty values where serialization or contracts care;
- JSON/protobuf/database representation mismatches;
- accidental breaking changes to exported APIs.

Pay special attention to code that "looks obvious" but relies on slice capacity,
map reference semantics, interface dynamic types, deferred execution, or goroutine timing.

---

# 3. Concurrency and goroutines

Concurrency findings are high-value. Inspect them deliberately.

## Goroutine lifecycle

For every newly introduced or materially changed goroutine, determine:

- who starts it;
- who owns it;
- how it stops;
- whether it observes cancellation;
- who waits for it;
- what happens on early return or error;
- whether it can outlive the request/service/object that created it;
- whether it can block forever;
- whether repeated calls create unbounded goroutines.

Flag goroutine leaks when a goroutine can remain blocked on:

- channel send/receive;
- network or I/O without deadline/cancellation;
- mutex/condition;
- timer/ticker;
- worker loop with no stop path.

Do not recommend a goroutine merely to make code "async".

## Channels

Check:

- clear ownership of channel creation and closing;
- only the sender/owner should normally close the channel;
- receivers must not close a channel they do not own;
- no send on closed channel;
- no double close;
- nil-channel behavior is intentional;
- buffered channel size has a concrete reason;
- unbuffered vs buffered semantics match the synchronization requirement;
- sends cannot block indefinitely when consumers exit;
- fan-in/fan-out terminates correctly;
- channel direction (`<-chan`, `chan<-`) is used when it clarifies ownership;
- channels are not used where a mutex or direct call would be simpler.

Do not enforce "channels everywhere". A mutex is often the clearer solution for shared state.

## Shared state

Check:

- maps are not concurrently read/written without synchronization;
- shared slices/struct fields are protected correctly;
- mutex coverage is consistent;
- locks are not copied after first use;
- code does not hold locks across slow network/disk calls without a concrete reason;
- lock ordering cannot deadlock;
- callbacks are not invoked under a lock unless intentionally safe;
- `defer mu.Unlock()` placement is correct;
- `RWMutex` is justified rather than assumed faster;
- atomics are used only when their memory/atomicity semantics are understood;
- compound invariants are not incorrectly implemented with independent atomics.

## WaitGroup / errgroup

Check:

- `Add` happens before work can call `Done`;
- every started unit decrements exactly once;
- no negative counter;
- no reuse while a previous `Wait` is active;
- failures/cancellation are propagated;
- `errgroup`-style cancellation does not leave producers blocked on sends.

## Timers and tickers

Check:

- tickers are stopped when lifetime is bounded;
- timers are reset/drained correctly when reused;
- loops do not repeatedly allocate timers unnecessarily;
- `time.After` is not used in hot or long-lived loops when it causes avoidable timer allocation/lifetime issues.

When concurrency is involved, run or recommend the race detector on the smallest relevant scope,
then broader scope when practical.

---

# 4. Context and cancellation

For request-scoped work:

- `context.Context` should normally be the first parameter;
- pass context through the call chain;
- do not store request contexts in structs;
- do not replace an available parent context with `context.Background()` or `TODO()` without reason;
- preserve cancellation and deadlines for HTTP, DB, gRPC, queue, and other blocking operations;
- call returned cancel functions when required;
- avoid context values for ordinary function parameters or dependency injection;
- context keys must avoid collisions and should not expose unsafe public key types;
- do not pass nil context;
- cancellation paths must not leak goroutines or resources.

A background task may intentionally detach from a request, but that decision must be explicit
and its lifetime must be owned by a longer-lived component.

---

# 5. Errors

Review error behavior as part of the public contract.

Check:

- errors are not silently discarded when callers need them;
- added context helps identify the failed operation;
- wrapping uses `%w` when callers are expected to inspect the underlying error;
- `errors.Is` / `errors.As` are used instead of brittle string comparison;
- errors are not matched by message text;
- sentinel and typed errors are used only when callers need programmatic classification;
- error messages are lower-case sentence fragments where appropriate and do not add redundant punctuation;
- the same error is not logged at every layer without a reason;
- code does not both log and return an error in a way that guarantees duplicate logs;
- HTTP/gRPC/domain error translation happens at the appropriate boundary;
- partial results plus errors have clear documented semantics;
- cleanup errors are handled where loss would matter;
- deferred cleanup does not accidentally overwrite or hide the primary error without intent.

Do not introduce custom error types merely for architecture aesthetics.

---

# 6. Interfaces and abstraction

Go interfaces describe required behavior; they are not class hierarchies.

Check:

- interfaces are defined close to the consumer when practical;
- interfaces are as small as the consumer actually needs;
- a concrete type is not hidden behind an interface without a testing, substitution, or boundary need;
- an interface is not created "for every service/repository" by habit;
- functions accept interfaces when they genuinely need multiple implementations;
- public APIs preferably return useful concrete types unless abstraction is part of the contract;
- one-method interfaces follow established naming where appropriate (`Reader`, `Writer`, etc.);
- implementations do not depend on compile-time assertions as a substitute for coherent design;
- mocks do not drive production abstractions into artificial shapes.

Flag Java/C#/TypeScript-style patterns when they add ceremony without Go-specific value, for example:

- `IUserService`, `UserServiceImpl`;
- `BaseService` / inheritance emulation through embedding;
- interfaces mirroring every method of a concrete implementation;
- deep generic repository hierarchies;
- getters/setters for plain data with no invariant;
- factories/builders when a struct literal or small constructor is clearer;
- dependency-injection containers where explicit construction is straightforward.

Do not reject an abstraction only because it resembles another ecosystem; reject it when
it worsens clarity, ownership, coupling, testability, or maintenance in this Go codebase.

---

# 7. Structs, methods, and receivers

Check receiver choices:

- pointer receiver when methods mutate the receiver, the value is large, contains synchronization primitives,
  or identity/reference semantics matter;
- value receiver when copy semantics are natural and consistent;
- avoid mixing pointer and value receivers without a clear reason;
- do not copy structs containing `sync.Mutex`, `sync.RWMutex`, `sync.Once`, atomics, or similar state after use.

Check constructors:

- do not require `NewX` when the zero value or struct literal is already correct and safe;
- use constructors when invariants, validation, defaults, unexported fields, or dependencies require them;
- ensure constructors return objects in valid states.

Check zero-value usability where idiomatic and practical.

Avoid excessive exported fields and exported symbols. Keep APIs smaller than implementation details.

---

# 8. Packages and dependencies

Review package boundaries, not only types.

Check:

- package names are short, clear, lower-case, and do not repeat caller-visible context;
- avoid vague packages such as `util`, `utils`, `common`, `misc`, `helpers` when they become dumping grounds;
- avoid package names such as `interfaces`, `models`, or `types` when grouping by technical category obscures domain ownership;
- avoid import cycles and suspicious dependency inversion created only to break them;
- avoid internal packages depending on higher-level delivery layers;
- keep `main` / wiring separate from reusable package behavior;
- do not expose internal implementation through unnecessary exports;
- use standard library functionality instead of dependencies when the standard solution is sufficient;
- new dependencies must justify maintenance, security, binary size, or operational cost;
- package `init()` must have a strong reason and must not hide critical application wiring.

Prefer package organization around cohesive responsibilities over framework-style layer folders
when the latter create scattered behavior.

---

# 9. HTTP, gRPC, DB, queues, and I/O

When applicable, inspect boundary behavior.

## HTTP

Check:

- request bodies are closed where required;
- clients have sensible timeouts or inherit request cancellation;
- default `http.Client` usage is intentional for long-running production calls;
- response status codes are validated;
- non-success response bodies are bounded before reading/logging;
- transports/clients are reused instead of recreated per request;
- handlers do not start unmanaged background goroutines;
- server timeouts and graceful shutdown are considered at the application boundary.

## gRPC

Check:

- context/deadlines propagate;
- status codes are mapped intentionally;
- streaming loops terminate on context/error;
- send/recv errors are handled;
- stream goroutines cannot leak;
- interceptors do not duplicate domain behavior.

## Database

Check:

- queries use context-aware APIs when relevant;
- rows/result resources are closed;
- `rows.Err()` is checked;
- transactions commit/rollback on all paths;
- network/DB calls are not performed while holding unrelated locks;
- N+1 queries or per-item round trips are identified where material;
- nullable DB semantics map correctly to Go values;
- scanning into reused objects does not alias unexpectedly.

## Queues/workers

Check:

- ack/nack/retry semantics;
- poison-message behavior;
- duplicate delivery and idempotency;
- cancellation and shutdown;
- worker limits/backpressure;
- goroutine growth;
- ordering assumptions;
- partial batch failures.

---

# 10. Performance and allocations

Performance review must be evidence-driven.

First ask whether the code is:

- on a hot path;
- processing large input;
- running at high frequency;
- latency-sensitive;
- allocation-sensitive;
- holding a lock;
- making external calls inside a loop.

Flag clear problems such as:

- repeated network/DB calls that can be batched;
- accidental O(n²) behavior on meaningful input sizes;
- repeated full scans;
- excessive conversions between `string` and `[]byte` in hot code;
- `fmt.Sprintf` in tight loops where simpler operations materially help;
- avoidable per-iteration allocations;
- growing slices without capacity when final size is cheaply known and meaningful;
- unnecessary copying of large structs/slices/buffers;
- reflection in performance-sensitive code without need;
- regex compilation inside repeated execution;
- repeated JSON encoder/decoder setup when a simpler streaming approach is appropriate;
- spawning unbounded goroutines for parallelism;
- contention introduced by broad locks;
- false "optimizations" that make code harder without measured benefit.

Do not report micro-optimizations in cold code as meaningful findings.

Before recommending a non-obvious optimization, prefer evidence from:

- benchmark (`go test -bench`);
- allocation benchmark (`-benchmem`);
- CPU/heap profile;
- trace;
- production metrics;
- clear complexity analysis.

Do not use "goroutines are faster" as a justification. Concurrency is not automatically parallelism
and parallelism is not automatically faster.

---

# 11. Idiomatic Go

Check common Go conventions.

## Naming

- use `MixedCaps` / `mixedCaps`, not underscores in Go identifiers;
- initialisms should be consistent with common Go usage (`ID`, `HTTP`, `URL`, `API`, etc.);
- avoid redundant names such as `user.UserService` when `user.Service` is clearer;
- method receivers should usually be short and consistent;
- local names may be short when scope is small;
- names farther from declaration should be more descriptive;
- avoid `GetX` for ordinary getters when `X()` is idiomatic.

Do not force short names where they reduce clarity.

## Control flow

Prefer:

- early returns over deeply nested `else`;
- simple `for`, `range`, and `switch`;
- direct error handling close to the failing operation;
- readable explicit code over clever one-liners.

Flag:

- unnecessary `else` after terminal `return`;
- huge functions with multiple unrelated responsibilities;
- excessive boolean flags controlling unrelated behavior;
- clever fallthrough/control flow that obscures invariants.

## `defer`

Check:

- cleanup is placed immediately after successful acquisition;
- deferred calls capture the intended values;
- defer inside large loops does not unintentionally retain resources until the outer function returns;
- performance concerns around `defer` are not raised without evidence on modern Go.

## Slices and maps

Check:

- nil vs empty semantics at serialization/public boundaries;
- preallocation only when size is known or profiling justifies it;
- map presence uses `v, ok := m[k]` when zero value is ambiguous;
- slices are not retained accidentally through tiny subslices of huge backing arrays;
- API ownership is clear when returning or storing mutable slices/maps;
- append does not violate aliasing expectations.

---

# 12. Generics

Use generics when they remove real duplication while preserving clarity and type safety.

Flag generics when:

- the abstraction is harder to understand than duplicated concrete code;
- type parameters exist only to imitate OOP hierarchies;
- constraints are unnecessarily broad or complex;
- reflection/interface code would actually be clearer;
- generic helpers obscure domain semantics.

Also flag missed generic reuse only when duplication is substantial and the generic abstraction is obvious,
stable, and useful to multiple callers.

Do not recommend generics merely because the language supports them.

---

# 13. Tests

Review tests for behavior, not line coverage theater.

Check:

- changed behavior has tests where practical;
- bug fixes include a regression test when possible;
- table-driven tests are used when they improve clarity;
- subtests have meaningful names;
- tests distinguish `got` from `want`;
- failure messages provide useful debugging context;
- helpers call `t.Helper()`;
- parallel tests do not share unsafe mutable state;
- `t.Cleanup` is used for test-scoped cleanup where appropriate;
- tests are deterministic and do not depend on arbitrary sleeps;
- time-sensitive tests use controllable clocks or bounded synchronization when warranted;
- temp files use test-managed temp directories;
- environment changes use test-scoped mechanisms where possible;
- mocks/fakes test meaningful contracts rather than exact implementation choreography;
- tests do not reimplement the production algorithm and therefore pass with the same bug;
- race-sensitive behavior is exercised when concurrency changed.

A missing test is a finding only when there is a concrete important behavior or regression risk to protect.

---

# 14. Logging and observability

Check:

- libraries do not impose global logging unexpectedly;
- logs contain useful operation/context identifiers without leaking secrets;
- errors are logged at an ownership/boundary layer instead of every layer;
- logging does not alter control flow correctness;
- high-frequency loops do not emit unbounded logs;
- retries expose enough information to diagnose repeated failures;
- metrics/tracing preserve request context where relevant.

Do not require logging for every error.

---

# 15. Security-sensitive Go issues

When relevant, check:

- path traversal and unsafe path joins;
- unbounded reads (`io.ReadAll`) from untrusted sources;
- decompression bombs / unbounded payload expansion;
- unsafe command construction;
- `html/template` vs `text/template` in HTML contexts;
- TLS verification is not disabled casually;
- secrets are not logged;
- crypto primitives use established standard-library or vetted implementations;
- random security tokens use cryptographic randomness;
- integer conversions cannot bypass size/limit checks;
- user-controlled regex or parsing cannot create obvious resource exhaustion;
- concurrency limits exist for externally triggerable expensive work.

Security findings must be tied to a realistic trust boundary.

---

# 16. Style and tooling

Mechanical formatting should be automated, not debated.

When tools are available, use repository configuration first.

Suggested checks:

```bash
gofmt -d .
go test ./...
go vet ./...
```

If the project uses it or it is already available:

```bash
staticcheck ./...
```

For concurrency-sensitive changes:

```bash
go test -race ./...
```

Prefer running the smallest affected package set first when the repository is large.

If the repository has `golangci-lint` configuration, use that configuration rather than inventing
a competing personal ruleset.

Do not fail review because an optional linter is not installed. Report tool unavailability separately
from code findings.

Respect the Go version declared in `go.mod`. Do not suggest syntax or APIs unavailable to that version.

`gofmt`/`goimports` own formatting. Do not create subjective formatting rules that fight them.

---

# 17. Evidence workflow

For every candidate issue:

1. identify the exact symbol/location;
2. reconstruct the relevant control/data flow;
3. inspect callers/callees or interface implementations when needed;
4. determine the concrete failure, maintenance cost, or measurable performance issue;
5. check whether tests or language/library guarantees already invalidate the concern;
6. assign severity only after understanding impact;
7. give the smallest reasonable fix direction.

Discard the finding if evidence does not survive this process.

For tricky language behavior, prefer official Go documentation/specification and standard-library behavior
over remembered folklore.

---

# 18. Severity

Use:

## BLOCKER

A confirmed issue that can cause severe production failure, data corruption/loss, major security impact,
system-wide deadlock, unrecoverable resource exhaustion, or a clearly broken public contract.

## HIGH

A confirmed correctness bug, race, goroutine/resource leak, deadlock path, broken cancellation,
serious API regression, transaction/idempotency defect, or significant production reliability problem.

## MEDIUM

A real maintainability/design/testability problem, non-idiomatic pattern with concrete cost,
performance issue with meaningful impact, weak error contract, avoidable coupling, or important missing test.

## LOW

A small but useful improvement with limited impact: naming, localized simplification,
minor API consistency, documentation, or style not already handled mechanically.

Do not inflate severity because a finding is easy to explain.

Style-only findings should almost never be HIGH.

---

# 19. Output format

Start with findings. Do not begin with praise or a generic summary.

For each finding:

```text
[HIGH] Short problem title

Location:
`path/file.go:line` or `package.Symbol`

Problem:
What is wrong.

Why it matters:
Concrete impact or broken invariant.

Evidence:
Relevant call path, language behavior, test/tool result, or reproduction scenario.

Recommended direction:
Smallest sensible correction. Do not provide a large refactor unless required.
```

Order findings by severity, then by impact.

After findings, optionally include:

```text
Verification:
- commands actually run;
- tests/lints/race checks and their results;
- important checks that could not be run.
```

If no meaningful findings remain after verification, output:

```text
CLEAN

Проверено:
- ...
```

Do not invent findings to avoid returning `CLEAN`.

---

# 20. Review anti-patterns

Never do these automatically:

- demand Clean Architecture / Hexagonal / DDD for a small package;
- add interfaces solely for mocking;
- split every function into helpers merely to reduce line count;
- introduce a repository/service layer by convention;
- recommend channels when a mutex/direct call is simpler;
- recommend mutexes when message passing already gives clear ownership;
- parallelize code without proving workload and lifecycle safety;
- replace readable loops with clever generic helpers;
- optimize allocations in cold code without evidence;
- demand constructors for every struct;
- demand getters/setters for plain fields;
- demand dependency injection frameworks;
- treat every exported function without a comment as a serious defect;
- turn `go vet`/linter warnings into a severity higher than their actual runtime impact;
- report existing unrelated problems unless they are directly required to understand the changed code;
- perform unrelated refactoring;
- change public contracts without explicit need.

The review should make the code more Go-like, not more ceremonious.

---

# 21. Go-specific red flags checklist

Before concluding, explicitly consider whether the change introduced any of these:

- goroutine with no clear stop/wait path;
- channel with unclear close ownership;
- concurrent map access;
- lock held during blocking I/O;
- copied mutex/atomic-containing value;
- cancellation dropped by replacing context;
- background context inside request-scoped work;
- response/body/rows/ticker/resource not closed;
- error ignored or converted to string matching;
- `%v` used where `%w` is required for error identity;
- log-and-return duplication;
- typed nil hidden in an interface;
- slice aliasing or unexpected backing-array mutation;
- pointer to loop/range value issue for the target Go version;
- unbounded goroutine fan-out;
- unbounded read or allocation from external input;
- transaction not closed on every path;
- retry without idempotency/backoff/limit where needed;
- per-item DB/network calls causing material N+1 behavior;
- unnecessary interface around one implementation;
- interface declared by provider instead of actual consumer need;
- package named `utils`, `common`, or `interfaces` becoming a dumping ground;
- non-idiomatic getter/setter or OOP emulation;
- benchmark-free micro-optimization that harms readability;
- arbitrary `time.Sleep` synchronization in tests;
- missing race-sensitive regression test after concurrency changes.

Only report items that are actually present and meaningful.
