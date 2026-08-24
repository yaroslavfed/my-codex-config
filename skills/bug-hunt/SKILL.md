---
name: bug-hunt
description: Системно расследует дефект: воспроизведение, call chain, root cause, аналогичные места и regression test. Не начинает с угадывания исправления.
---

# Bug Hunt

Используй при расследовании существующего дефекта.

## Правило

Не начинай с изменения кода.

Сначала докажи root cause.

## Workflow

### 1. Зафиксировать симптом

Определи:

- фактическое поведение;
- ожидаемое поведение;
- входные данные;
- окружение;
- минимальный сценарий воспроизведения.

Если информации не хватает, сначала попробуй получить её из кода, тестов, логов и конфигурации.

### 2. Reproduce

Предпочтительно получить один из вариантов:

- падающий unit test;
- падающий integration/e2e test;
- воспроизводимую run configuration;
- детерминированную последовательность действий.

### 3. Trace

Через IDE/MCP проследи:

`entry point → controller/handler → service → persistence/external client → output`

Найди:

- definitions;
- usages;
- transformations;
- intermediate state;
- error handling;
- branching.

### 4. Root cause

Сформулируй root cause одним конкретным утверждением.

Плохой пример:

> Возможно проблема связана с таймзоной.

Хороший пример:

> `startOf('day')` вычисляется в timezone процесса до применения timezone пользователя, поэтому записи после 20:00 UTC попадают в следующий локальный день.

### 5. Blast radius

Найди аналогичные места:

- usages той же функции;
- копии той же логики;
- другие команды/handlers;
- общие utilities;
- фоновые jobs.

### 6. Regression test

До исправления опиши тест, который:

- воспроизводит конкретный дефект;
- падает на старой реализации;
- проходит после исправления;
- не привязан к внутренним деталям без необходимости.

### 7. Fix proposal

Только после подтверждения root cause предложи минимальное исправление.

Не делай архитектурный rewrite, если дефект исправляется локально и безопасно.

## Результат

Выдай:

1. Symptom
2. Reproduction
3. Execution path
4. Root cause
5. Blast radius
6. Regression test
7. Minimal fix
8. Risks / что проверить после исправления

Если root cause не доказан, явно напиши:

`ROOT CAUSE NOT CONFIRMED`

## Language

All user-facing output must be in Russian.