---
name: review-fixes
description: Investigate, implement, verify, and respond to existing code review comments. Use when the user provides review feedback and asks to fix, investigate, plan, or prepare concise responses to the comments.
---

# Review Fixes

Use this skill when the user provides existing code review comments and asks to investigate them, fix them, prepare fixes, or respond to the review.

Typical triggers include:

- "Давай исправим замечания по ревью"
- "Вот комментарии ревьюера, исправь их"
- "Разбери замечания и подготовь ответы"
- "Исправь review comments"
- "Проверь эти замечания и внеси необходимые изменения"
- "Давай отработаем замечания ревью"
- "Составь план исправления замечаний ревью"

## Goal

For every review comment:

1. understand what the reviewer is asking;
2. investigate the relevant implementation and surrounding behavior;
3. determine whether the comment is valid and applicable;
4. decide whether it should be fixed, partially fixed, or left unchanged;
5. implement the appropriate fix when implementation is allowed;
6. verify the resulting behavior;
7. provide a concise response describing the outcome.

Do not blindly accept review feedback.

A review comment is a hypothesis that must be verified against the actual codebase, contracts, architecture, tests, usages, and surrounding behavior.

The goal is not to satisfy the wording of the reviewer mechanically. The goal is to resolve the actual problem, if one exists, without introducing unnecessary changes.

## General Rules

- Process every review comment provided by the user.
- Preserve the original order of comments in the final response.
- Do not silently skip comments.
- Investigate before modifying code.
- Verify the reviewer's assumptions against the actual implementation.
- Prefer the smallest correct change that addresses the real issue.
- Do not expand the task into unrelated refactoring.
- Do not introduce new abstractions unless they are necessary for the fix.
- Preserve existing public contracts unless changing them is explicitly required and justified.
- Check usages and surrounding behavior before modifying shared code.
- Consider existing architecture and project conventions.
- Reuse existing abstractions and patterns when appropriate.
- Do not change unrelated formatting or code style merely while touching a file.
- Verify fixes using the most relevant available tests, type checks, lint checks, inspections, or other appropriate validation.
- Do not claim that a comment is fixed unless the resulting behavior has been verified to a reasonable extent.
- User-facing review responses must be in Russian unless the user explicitly requests another language.

## Investigation

Before deciding how to handle a review comment:

1. Locate the code referenced directly or indirectly by the comment.
2. Inspect the surrounding implementation.
3. Check relevant callers, usages, models, contracts, DTOs, repositories, tests, and configuration when they affect the conclusion.
4. Determine what behavior exists before the change.
5. Determine whether the problem described by the reviewer can actually occur.
6. Determine the intended behavior based on the codebase rather than assumptions.
7. Only then decide whether a code change is necessary.

Do not infer that a review comment is correct merely because it sounds reasonable.

If a comment is ambiguous, investigate the most likely intended concern from the surrounding code and review context. Do not invent a broader requirement than the reviewer actually raised.

## Possible Outcomes

Every review comment must end in one of these outcomes.

### Fixed

Use when the review comment is valid and the implementation was changed accordingly.

"Fixed" means:

- the problem was investigated;
- an appropriate change was made;
- the resulting behavior was verified to a reasonable extent.

A code edit by itself is not enough to mark a comment as fixed.

### Partially fixed

Use when the underlying concern is valid but only part of the requested or suggested change is appropriate.

Apply the justified part.

Do not implement the rest merely for compliance if it is unnecessary, incorrect, harmful, or outside the justified scope.

The final response must briefly explain what was addressed and why the remaining part was not implemented.

### Not fixed

Do not change code merely to satisfy the wording of a review comment.

A comment may remain unfixed when, for example:

- the reported problem does not exist;
- the code already handles the described case;
- the proposed change conflicts with the actual contract;
- the proposed change would introduce a regression;
- the comment is based on an incorrect assumption;
- the suggested implementation is unnecessary even though the broader concern is understandable;
- the behavior is intentional and supported by the surrounding design;
- the issue belongs outside the current task scope and cannot be safely mixed into the current fix.

The reason must be established from the codebase or task context rather than assumed.

When declining a review comment, explain the technical reason concisely and confidently without becoming argumentative.

## Implementation Mode

When code modifications are allowed, process each review comment using this workflow:

1. Read and understand the comment.
2. Locate the relevant implementation.
3. Inspect surrounding code and usages.
4. Verify the reviewer's assumption.
5. Determine the smallest correct solution.
6. Implement the fix if necessary.
7. Run relevant verification.
8. Re-check the original review comment against the resulting implementation.
9. Record the final outcome for the response.

Do not stop after changing the code.

The final check should answer:

- Does the original problem still exist?
- Did the fix introduce a new problem?
- Does the resulting behavior match the intended contract?
- Can the comment now be answered accurately?

When several review comments affect the same code, investigate them together when useful, but still provide a separate final response for every original comment.

## Plan Mode

When operating in Plan mode:

- do not modify code;
- perform the same investigation required in implementation mode;
- verify whether every review comment is valid;
- determine the intended resolution for each applicable comment;
- identify comments that should not be implemented and explain why;
- identify important dependencies or risks that affect the fix;
- keep the plan focused on resolving the supplied review comments.

The plan must be based on inspected code, not merely on the text of the review comments.

Do not present speculative implementation details as facts.

In Plan mode, the final status wording should reflect that changes have not yet been applied.

Prefer wording such as:

- "Требует исправления."
- "Исправление запланировано."
- "Частично применимо."
- "Изменение не требуется."

Do not write "Исправлено" when no code was modified.

## Verification

Use verification proportional to the change.

Prefer the most focused checks that can demonstrate the behavior:

- relevant unit tests;
- relevant integration tests;
- targeted test files;
- type checking;
- linting;
- build checks;
- inspection of affected usages;
- existing tests that demonstrate the relevant contract.

Do not run an unnecessarily broad validation suite when a focused check is sufficient, unless project instructions require it.

If verification cannot be completed, do not hide this fact.

A comment may still be implemented, but the final response should briefly state the relevant verification limitation when it materially affects confidence.

Do not turn verification details into a long test report unless the user asks for one.

## Final Response

The final response must contain an answer for every review comment.

Use the original review comments as the primary structure and preserve their order.

For every comment state:

- its outcome;
- what the resulting behavior is now;
- when not fixed or only partially fixed, why;
- when relevant, any important verification limitation.

Keep responses concise.

For a small comment, usually 1-2 sentences are enough.

For a large comment or a comment containing multiple meaningful subproblems, briefly cover each important part while keeping the response compact.

The response should help the user paste or adapt the text into a code review discussion without first translating a technical changelog into normal language.

## Response Style

### Describe the outcome, not the edit history

The response must be written as short connected prose.

Do NOT turn the answer into a list of implementation edits.

Bad:

"Исправлено:
- заменён вызов X;
- добавлена проверка Y;
- изменён метод Z."

Bad:

"Исправлено. Сделано следующее:
1. Добавлена проверка.
2. Изменён repository.
3. Обновлён service."

Bad:

"Исправлено. В `foo.service.ts` поменял условие, в `bar.repository.ts` добавил метод, а в тестах обновил mock."

Good:

"Исправлено. Теперь значение проверяется до обращения к repository, поэтому некорректный запрос завершается без обращения к БД, а поведение валидного сценария осталось прежним."

The important information is:

- what problem was resolved;
- whether the concern was valid;
- how the behavior works now.

Implementation details may be mentioned when they are necessary to explain the result, but they must be integrated into connected prose rather than presented as a changelog.

### Do not over-explain

Avoid:

- long descriptions of every edited file;
- line-by-line summaries;
- exhaustive lists of tests;
- repeating the full implementation;
- narrating obvious mechanical changes;
- explaining basic language or framework concepts unless they are essential to the review response.

The user asked for concise review responses, not an implementation diary.

### Multi-part comments

If one review comment contains several meaningful sub-comments, each important part must be addressed.

Do not flatten a partially resolved multi-part comment into a misleading single "Исправлено".

Prefer a short connected explanation that makes the status of each meaningful part clear.

If needed for readability, short paragraphs may be used inside the response to one large comment.

Do not convert the technical resolution into a bullet list of edits.

## Recommended Final Format

Use a compact structure such as:

### Замечание 1 — Исправлено

Короткий связный ответ о том, что изменилось в поведении и почему замечание теперь закрыто.

### Замечание 2 — Не исправлено

Короткое объяснение, почему изменение не требуется или почему предложение ревьюера не соответствует фактическому контракту.

### Замечание 3 — Частично исправлено

Короткое объяснение, какая часть проблемы была реальной и устранена, а какая часть предложенного изменения не была применена и почему.

The headings are organizational labels for the original review comments.

They must not be followed by changelog-style edit lists.

If the reviewer supplied explicit numbering or titles, prefer retaining those identifiers.

## Examples

### Example: small valid comment

Review comment:

"Здесь возможен повторный insert при повторной обработке одного события."

Response:

### Замечание 1 — Исправлено

Теперь повторная обработка того же события переиспользует существующую запись и не создаёт дубликат, поэтому операция остаётся идемпотентной.

### Example: incorrect assumption

Review comment:

"Нужно обработать `expiresAt === null`, иначе здесь будет ошибка."

Response:

### Замечание 2 — Не исправлено

`expiresAt` по действующему контракту не может быть `null`: поле обязательное в модели, а все пути создания записи его задают. Дополнительная ветка здесь не устраняет реальный сценарий и только дублировала бы уже гарантированный инвариант.

### Example: partially valid suggestion

Review comment:

"Нужно обработать ошибку и заодно вынести всю эту логику в отдельный сервис."

Response:

### Замечание 3 — Частично исправлено

Проблема с обработкой ошибки действительно существовала и устранена, поэтому сбой теперь корректно обрабатывается в текущем сценарии. Перенос всей логики в новый сервис не выполнялся, поскольку для исправления дефекта он не требуется и необоснованно расширил бы область изменений.

### Example: large multi-part comment

Review comment:

"Здесь три проблемы: повторный запрос создаёт дубликат, ошибка внешнего сервиса проглатывается, а метод слишком большой и его нужно разбить на четыре приватных метода."

Response:

### Замечание 4 — Частично исправлено

Дубликаты при повторном запросе устранены, а ошибка внешнего сервиса теперь сохраняет исходную семантику ошибки вместо успешного продолжения обработки. Дополнительное дробление метода на четыре приватных метода не выполнялось: после исправления дефектов метод остаётся линейным и читаемым, а такое разбиение не решает отдельную проблему и добавило бы только механическую фрагментацию.

## Completion Criteria

The task is complete only when:

- every supplied review comment has been investigated;
- every comment has an explicit outcome;
- applicable fixes have been implemented when implementation is allowed;
- relevant verification has been performed where possible;
- no comment has been silently ignored;
- the final response describes resulting behavior rather than listing edits;
- the final response is concise enough to be useful as a review reply.
