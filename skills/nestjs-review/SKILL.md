---
name: nestjs-review
description: "Проверяет NestJS-специфичные архитектурные и runtime-проблемы: DI, modules, scopes, lifecycle, controllers, providers, guards, interceptors и workers."
---

# NestJS Review

Используй как специализированную проверку NestJS-кода.

## DI / Modules

Проверь:

- provider экспортируется только когда это действительно нужно;
- injection token совпадает;
- нет случайного создания двух экземпляров stateful provider;
- scopes (`DEFAULT`, `REQUEST`, `TRANSIENT`) соответствуют ожиданиям;
- нет скрытой circular dependency;
- `forwardRef` не маскирует архитектурную проблему.

## Controllers

Controllers должны:

- валидировать/принимать transport input;
- передавать работу в application/service layer;
- преобразовывать результат в transport response.

Отмечай бизнес-логику в controller только если она реально усложняет тестирование/повторное использование/контракты.

## Providers / Services

Проверь:

- service не смешивает transport, persistence и domain logic без причины;
- ошибки внешних клиентов преобразуются осознанно;
- provider не хранит per-request mutable state при singleton scope;
- lifecycle hooks корректны.

## Background jobs / workers

Особенно проверяй:

- overlapping execution;
- duplicate processing;
- retry;
- idempotency;
- lock strategy;
- shutdown;
- long-running promise;
- failure после частичного side effect.

## HTTP clients

Проверь:

- timeout;
- retries;
- cancellation/abort, если релевантно;
- 401/403/404/409/429/5xx;
- response validation;
- отсутствие утечки credentials/token в logs.

## Validation / DTO

Проверь:

- runtime validation соответствует TypeScript типам;
- optional/nullable semantics согласованы;
- преобразование query/path/body;
- enum/union значения;
- class-transformer/class-validator side effects, если используются.

## Exceptions

Проверь:

- Nest HttpException используется только там, где transport semantics действительно уместны;
- domain/service слой не становится жёстко связан с HTTP без необходимости;
- catch не скрывает root cause;
- logging не дублируется на каждом слое.

## Tests

Проверь:

- unit tests не мокают весь NestJS без необходимости;
- integration tests проверяют wiring, когда именно wiring является риском;
- e2e тестирует реальный transport contract.

## Результат

Используй стандартный формат findings из `code-review`.

Если NestJS-специфичных проблем нет:

`NESTJS REVIEW: CLEAN`

## Language

All user-facing output must be in Russian.
