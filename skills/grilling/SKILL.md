---
name: grilling
description: Use when the user wants to stress-test a plan, architecture, decision, or idea before implementation. Resolve the decision tree one dependency-aware question at a time and provide a recommended answer for every question.
disable-model-invocation: true
---

# Grilling

Interview the user until important design decisions are explicit and there are no material silent assumptions.

## Rules

- Ask one decision question at a time.
- For every question, give your recommended answer and the reasoning behind it.
- Resolve prerequisites before asking dependent questions.
- If a fact can be learned from the codebase, repository, tools, documentation, or environment, investigate it yourself instead of asking the user.
- Ask the user only for decisions, priorities, business intent, or unavailable facts.
- Challenge contradictions and vague answers instead of silently choosing an interpretation.
- Do not implement the plan during the grilling session.

## Decision tree

Treat the design as a tree of decisions.

At each step:

1. Identify the next unresolved decision whose prerequisites are already known.
2. Ask that question.
3. Present realistic options when useful.
4. Recommend one option.
5. Wait for the user's decision.
6. Recompute which decisions are now unblocked.

Do not ask a downstream question whose answer depends on an unresolved upstream decision.

## Good targets

Focus on decisions that materially affect implementation, such as:

- source of truth
- ownership of state
- API or event contract
- consistency requirements
- transaction boundary
- idempotency
- authorization
- compatibility
- failure/retry behavior
- persistence model
- sync strategy
- caching
- rollout/migration

Avoid generic questions that do not change the design.

## Completion

When no material decision remains unresolved, summarize:

- agreed decisions
- rejected alternatives worth remembering
- remaining assumptions or external unknowns

Do not start implementation until the user explicitly moves to implementation.
