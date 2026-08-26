---
name: codebase-design
description: Use when designing or improving a module boundary, interface, seam, or internal architecture. Compare materially different designs and prefer deep modules with small caller-facing contracts, hidden complexity, good locality, and testable seams.
disable-model-invocation: true
---

# Codebase design

Design modules that hide substantial coherent behavior behind a small, stable caller-facing interface.

## Vocabulary

- **Module**: a cohesive unit with an interface and implementation; scale may be a function, class, package, or vertical slice.
- **Interface**: everything callers must know, including methods/types, invariants, ordering, errors, configuration obligations, and relevant performance characteristics.
- **Seam**: a place where behavior can vary without editing the caller.
- **Adapter**: a concrete implementation used at a seam.
- **Depth**: how much useful behavior is provided for how little interface callers must understand.
- **Locality**: how well related behavior, knowledge, bugs, and verification stay concentrated instead of leaking into many callers.

## Design process

### 1. State responsibility

Define in one sentence what the module owns and explicitly what it does not own.

### 2. Identify callers

List representative callers and the behavior they need. Do not design an interface around imagined consumers with no evidence.

### 3. Design it more than once

Produce 2-4 materially different interface/module shapes when the design is non-trivial.

Alternatives must differ in real architecture, for example:

- where orchestration lives
- what state is owned
- where the seam is placed
- whether behavior is command-oriented or domain-oriented
- what callers need to know

Do not produce cosmetic variants of the same design.

### 4. Compare alternatives

Evaluate each option against:

- interface size and cognitive load
- hidden complexity
- locality of behavior
- coupling to infrastructure
- testability through the same interface callers use
- failure semantics
- compatibility/migration cost
- fit with existing repository patterns

### 5. Prefer earned seams

Do not introduce abstractions merely because they are theoretically swappable.

A seam should be justified by actual variation, isolation needs, testing needs, ownership, or architectural boundaries.

Avoid chains of pass-through services/repositories that add names without hiding complexity.

### 6. Recommend one design

Show:

- proposed interface/contract
- what it hides
- dependencies/adapters
- why it fits the existing codebase
- tradeoffs
- what would make you choose another option

Do not implement unless separately asked.
