---
name: token-efficiency
description: >
  Modifier skill for reducing token usage during agentic coding work without sacrificing correctness.
  Use only when explicitly invoked or combined with another skill. Minimize unnecessary repository reads,
  repeated explanations, redundant tool calls, oversized outputs, and broad verification while preserving
  enough evidence to complete the task safely and correctly.
---

# Token Efficiency

This is a modifier skill. It changes how the agent spends context and tokens; it does not replace the workflow,
quality bar, or correctness requirements of the primary skill or `AGENTS.md`.

## Priority

Optimize in this order:

1. correctness
2. sufficient evidence
3. minimal scope
4. token efficiency

Never save tokens by guessing, skipping a required verification, hiding uncertainty, or ignoring relevant code.

## Core principle

Use the smallest amount of context that is sufficient for the next decision.

Prefer:

`search -> targeted read -> change -> targeted verification`

Avoid:

`read everything -> summarize everything -> change everything -> run everything`

## 1. Repository exploration

Start from the narrowest useful operation:

1. locate the relevant symbol, file, route, class, method, test, or usage
2. inspect the smallest relevant code region
3. expand outward only when the current evidence is insufficient
4. read a complete file only when understanding the whole file is materially necessary
5. inspect neighboring modules or architectural layers only when dependencies require it

Prefer symbol/usages/reference search through IDE/MCP or repository search over opening many files manually.

Do not explore unrelated code merely to build general repository context.

Stop exploration when there is enough evidence to make the next implementation or review decision.

## 2. Avoid redundant reads

Do not reread unchanged content unless:

- a later discovery changes its interpretation
- the exact code is needed for an edit
- verification requires checking the final state
- the previous read was incomplete

Keep track of already established facts during the current task and reuse them.

Do not repeatedly search for the same symbol or usage unless the code changed or the previous result was incomplete.

## 3. Progressive context expansion

When context is uncertain, expand progressively:

1. symbol or exact search result
2. small surrounding region
3. relevant method/class
4. complete file
5. directly connected files
6. broader subsystem

Do not jump directly to steps 4-6 without a reason.

## 4. Tool-call discipline

Every tool call should answer a concrete question or perform a concrete action.

Before a broad search/read, prefer a narrower query if it can answer the same question.

Batch independent searches when supported and when batching does not produce excessive irrelevant output.

Do not call multiple tools to confirm the same trivial fact unless confidence is genuinely insufficient.

When a tool returns enough evidence, continue the task instead of collecting redundant confirmation.

## 5. Implementation scope

Make the smallest coherent change that satisfies the task.

Do not:

- refactor unrelated code
- introduce abstractions without a demonstrated need
- rename unrelated symbols
- clean up neighboring code opportunistically
- change public contracts unless required

Reuse existing project patterns when they are adequate.

Smaller diffs reduce both implementation risk and the context required for review.

## 6. Explanations

Do not narrate routine tool usage.

Avoid messages such as:

- "Now I will inspect the repository..."
- "Next I am going to open the file..."
- "I found the file and will now analyze it..."

Explain reasoning when it affects a decision, exposes uncertainty, teaches something requested by the user,
or is necessary to understand a trade-off.

Do not restate the task unless clarification is needed.

Do not repeatedly restate the plan. Mention plan changes only when they materially affect the work.

## 7. Code output

When the agent edits files directly, do not reproduce large changed files in chat.

Report:

- affected files
- important behavioral changes
- relevant caveats
- verification result

Show code snippets only when they help explain a non-obvious decision, the user explicitly asks for them,
or the agent cannot edit the file directly.

Prefer the smallest useful snippet.

## 8. Verification

Use the narrowest verification that provides sufficient confidence.

Prefer, when appropriate:

1. affected unit test
2. affected package/module tests
3. targeted typecheck/lint/build
4. broader test suite

Escalate verification when:

- the change affects shared contracts
- the change crosses module/service boundaries
- targeted verification fails
- project rules require a broader check
- the risk of regression justifies it

Do not skip mandatory project checks merely to save tokens.

Avoid dumping long successful command output. Report the command/check and result concisely.

For failures, preserve the relevant error details needed for diagnosis.

## 9. Reviews and investigations

For review, debugging, bug-hunt, and investigation tasks:

- search from the reported behavior or changed code outward
- prioritize executable paths and actual usages
- distinguish proven findings from hypotheses
- stop following a hypothesis once evidence disproves it
- avoid cataloging unrelated observations
- report only actionable findings unless the user asks for exhaustive analysis

Do not reduce review depth merely to reduce output. Reduce irrelevant exploration instead.

## 10. Interaction with other skills

This skill modifies another workflow rather than replacing it.

Examples:

`$bug-hunt $token-efficiency`

`$code-review $token-efficiency`

`$learning-mode $token-efficiency`

When another skill requires an action, keep that action unless it is purely redundant.

If another skill asks for detailed explanation, provide the detail needed by that workflow but avoid duplication.

For `learning-mode` specifically:

- preserve explanations that have learning value
- avoid repeating previously explained concepts
- keep mechanical steps terse
- reuse established terminology and context
- do not reveal future steps before they become relevant
- prefer one focused explanation over several paraphrases of the same idea

## 11. Context pressure

If the task becomes large:

1. preserve decisions, constraints, unresolved questions, and verified facts
2. discard conversational narration and obsolete hypotheses first
3. avoid carrying large raw outputs when a concise factual summary is sufficient
4. reopen exact source code when precision is required rather than relying on an uncertain summary

Do not compress away information required for correctness.

## 12. Final response

Keep the final response proportional to the task.

For ordinary implementation work, prefer a compact structure containing:

- what changed
- verification performed
- remaining issue or caveat, only if one exists

Do not provide a chronological diary of the work.

For reviews, preserve all substantive findings even if the final response becomes longer.

## Anti-goals

Token efficiency does not mean:

- fewer tests regardless of risk
- shallow code review
- guessing instead of reading code
- ignoring edge cases
- suppressing relevant errors
- avoiding necessary documentation
- minimizing the answer when the user explicitly requested detail

The goal is to eliminate waste, not evidence.
