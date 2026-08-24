---
name: adversarial-review
description: Агрессивно пытается опровергнуть корректность решения, моделируя production-сбои, плохие входные данные, конкуренцию и нарушение предположений. Все findings должны подтверждаться доказательствами.
---

# Adversarial Review

Используй после обычной реализации или перед MR для поиска скрытых дефектов.

## Главное правило

Твоя задача — попытаться доказать, что решение сломано.

Но:

> отсутствие найденного дефекта является допустимым результатом.

Не создавай искусственные замечания.

## Фазы

### 1. Reconstruction

Сначала восстанови:

- какую проблему решает изменение;
- какие предположения делает реализация;
- какие данные считаются валидными;
- какие внешние системы участвуют;
- какие состояния считаются невозможными.

Не критикуй до понимания модели.

### 2. Attack

Попытайся сломать решение по направлениям:

#### Boundary cases
- пустые коллекции;
- `null` / `undefined`;
- одинаковые timestamps;
- очень старые/будущие даты;
- DST/timezone;
- duplicate IDs;
- partial data;
- malformed external responses.

#### State transitions
- создание → изменение → удаление;
- повторное выполнение;
- операция после revoke/disconnect;
- stale state;
- entity исчезла из bounded query, но не была удалена;
- восстановление после временного сбоя.

#### Concurrency
- два worker одновременно;
- duplicate message delivery;
- retry после частичного успеха;
- lock истёк во время выполнения;
- один пользователь/ресурс обрабатывается несколькими путями.

#### External systems
- timeout;
- 401/403/404/409/429/500;
- частичный ответ;
- stale ETag;
- pagination;
- reordered responses;
- network failure после успешной удалённой операции.

#### Persistence
- операция между двумя DB writes;
- transaction rollback;
- unique constraint;
- несовпадение DB state и external state;
- migrations / backward compatibility.

#### NestJS runtime
- provider scope;
- lifecycle;
- shutdown;
- scheduler overlapping;
- exception filters;
- interceptor side effects.

### 3. Critic pass

Для каждого найденного finding попытайся сам его опровергнуть.

Вопросы:

- действительно ли этот сценарий достижим;
- есть ли уже защита в другом месте;
- нарушает ли это реальный контракт;
- может ли тест/тип/constraint исключить проблему;
- это defect или только preference.

Finding остаётся только если пережил этот этап.

### 4. Evidence verification

Используй IDE/MCP:

- usages;
- definitions;
- inspections;
- tests;
- run configurations;
- git diff;
- terminal только при необходимости.

Не основывай P0/P1 на предположении без подтверждения.

## Формат

### Surviving findings

**[P1] ...**

- Assumption under attack:
- Evidence:
- Failure scenario:
- Impact:
- Why current protection is insufficient:
- Suggested direction:

### Rejected hypotheses

Кратко перечисли 1–5 наиболее правдоподобных гипотез, которые были проверены и не подтвердились.

### Verdict

Один из:

- `BLOCK`
- `NEEDS_FIXES`
- `ACCEPTABLE_WITH_RISK`
- `CLEAN`

## Запрещено

- оценивать код по вкусу;
- требовать архитектурный rewrite без доказанного риска;
- считать "может быть" достаточным основанием;
- автоматически предлагать больше слоёв/абстракций;
- исправлять код, если пользователь просил только review.

## Language

All user-facing output must be in Russian.