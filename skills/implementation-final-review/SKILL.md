---
name: implementation-final-review
description: Финальная проверка законченной реализации перед MR/merge: полный diff, контракты, тесты, runtime-поведение и regression coverage.
---

# Implementation Final Review

Используй, когда реализация считается завершённой.

Это gate перед MR/merge, а не brainstorming.

## 1. Scope

Определи полный diff относительно merge-base.

Проверь не только изменённые строки, но и:

- новые/удалённые файлы;
- migrations;
- package.json / lockfile;
- env/config;
- DTO/contracts;
- tests;
- CI/runtime scripts.

## 2. Completeness

Сопоставь реализацию с задачей.

Проверь:

- все ли обязательные сценарии реализованы;
- нет ли TODO/stub/temporary logic;
- нет ли feature path, который остался на старой модели;
- cleanup старого кода выполнен там, где необходим.

## 3. Runtime correctness

Проверь:

- async flow;
- lifecycle;
- shutdown;
- resource cleanup;
- retries;
- concurrency;
- idempotency;
- DB transactions;
- external API failures.

## 4. Contracts

Особое внимание:

- REST/GraphQL/gRPC DTO;
- RabbitMQ/event payloads;
- database constraints;
- public service methods;
- config/env variables;
- dates/timezones;
- backwards compatibility.

## 5. NestJS boundaries

- modules;
- providers;
- scopes;
- injection tokens;
- guards/interceptors/filters;
- scheduler/worker overlap;
- circular dependencies.

## 6. Dependency check

Если добавлен package:

- действительно ли он нужен;
- используется ли поддерживаемая версия;
- не дублирует ли существующую зависимость;
- есть ли runtime/import последствия;
- lockfile изменён ожидаемо.

## 7. Validation

Через IDE/MCP запусти максимально релевантный набор:

1. IDE inspections;
2. typecheck/build;
3. lint;
4. unit tests;
5. integration/e2e tests, относящиеся к изменению.

Не запускай опасные команды без необходимости.

## 8. Final verdict

### Blocking findings
Только реальные P0/P1.

### Non-blocking findings
P2/P3, только если полезны.

### Verification
Что реально было запущено/проверено.

### Unverified
Что проверить не удалось.

### Verdict

Один из:

- `READY`
- `READY_WITH_NOTES`
- `NOT_READY`

`READY` допустим. Не нужно находить проблему ради оправдания review.

## Language

All user-facing output must be in Russian.