---
name: nestjs-review
description: "Специализированный pre-MR review NestJS/TypeScript-кода. Проверяет DI/modules/scopes/lifecycle, async control flow, controllers/services/repositories, guards/interceptors, workers/cron, ConfigService, Sequelize/TypeORM transactions, runtime contracts и NestJS-специфичные test gaps. Используй вместе с senior-review: этот skill не заменяет общий поведенческий и mutation review."
---

# NestJS Review

## Назначение

Используй этот skill как специализированный второй проход после общего `senior-review` либо самостоятельно, если задача почти целиком связана с NestJS wiring/runtime.

Не дублируй общий review без необходимости.

Главная цель здесь — найти дефекты, характерные именно для NestJS/TypeScript runtime:

- DI и module wiring;
- scope/lifecycle;
- async/Promise ошибки;
- неправильные границы controller/service/repository;
- background jobs;
- configuration;
- transaction/locking semantics;
- расхождение TypeScript-типа с runtime-данными;
- NestJS-специфичные ложноположительные тесты.

Все пользовательские ответы выдавай на русском.

---

# 1. DI / Modules

Проверь:

- provider объявлен в нужном module;
- provider экспортируется только если нужен снаружи;
- импортирующий module действительно импортирует module-поставщик;
- injection token совпадает с provider token;
- interface token не перепутан с concrete class;
- нет случайного создания двух экземпляров stateful provider;
- нет дублирующей регистрации одного provider в нескольких modules;
- scope (`DEFAULT`, `REQUEST`, `TRANSIENT`) соответствует ожидаемому lifetime;
- singleton не хранит mutable per-request/per-user state;
- `forwardRef` не скрывает архитектурную циклическую зависимость;
- optional injection не превращает обязательную зависимость в silent fallback;
- dynamic module конфиг не регистрируется дважды с разными значениями.

При изменении provider обязательно просмотри usages и module wiring, а не только его класс.

---

# 2. Lifecycle и startup radius

Особенно проверяй:

- `onModuleInit`;
- `onApplicationBootstrap`;
- constructors с side effects;
- initial sync;
- startup migrations/bootstrap data;
- scheduled registration.

Для каждого внешнего payload или DB-read, выполняемого во время startup, спроси:

> Может ли валидное, но частично заполненное production-значение уронить весь запуск приложения?

Ищи:

- `.trim()` на nullable/optional поле;
- незащищённый parse;
- сетевой запрос без корректной обработки;
- обязательную конфигурацию с silent default;
- background initialization, которая должна деградировать, но вместо этого валит процесс.

Если ошибка в lifecycle hook делает приложение неготовым, severity оценивай по реальному startup impact.

---

# 3. Async / Promise correctness

Обязательно проверяй:

- пропущенный `await`;
- Promise, используемый в `if`;
- async predicate в `filter/find/some/every` без корректного ожидания;
- fire-and-forget Promise без явного намерения;
- потерянную ошибку;
- `void` вызов без причины;
- async callback внутри transaction/loop, который не await-ится;
- исключение, выброшенное после частичного side effect.

Особенно опасный паттерн:

```ts
if (this.validateAccess(...)) {
  ...
}
```

если `validateAccess` возвращает `Promise<boolean>`.

Проверяй не только типы, но и фактическую runtime-семантику.

---

# 4. Controllers

Controller должен в основном:

- принять transport input;
- вызвать application/service layer;
- вернуть/сформировать transport response.

Ищи:

- бизнес-правила в controller;
- прямые repository calls;
- транзакции в controller без необходимости;
- повторную normalization/validation после уже установленного DTO contract;
- расхождение REST/GraphQL/WebSocket путей одного use case;
- transport-specific exception в глубоком domain/service слое без причины.

Не требуй формального "thin controller", если текущая логика проста и не создаёт конкретной проблемы.

---

# 5. Services / responsibility boundaries

Проверь:

- один и тот же бизнес-предикат не реализован в нескольких services/workers/guards;
- service не смешивает persistence, transport и domain logic без причины;
- formatter/normalizer/policy имеет одного владельца;
- одинаковые правила timezone/authorization/identity не копируются по consumers;
- service не получает уже "resolved" значение и затем повторно применяет default;
- public method не держится только тестами;
- optional аргумент не является опасным test-only default.

Если параметр production-пути обязателен, предпочитай обязательный TypeScript contract вместо default, существующего только ради тестов.

---

# 6. Guards / Authorization

При изменении ролей, capabilities, department/tree logic или impersonation:

- найди все guards;
- найди manual/admin commands;
- найди background sync, способный перезаписать capability;
- проверь scope: direct department vs subtree vs global;
- проверь docs/templates, которые зависят от capability;
- проверь, не стала ли одна boolean-колонка косвенно выдавать слишком широкий доступ.

Проверь, что guard и handler не загружают одного пользователя повторно без необходимости.

Если User уже загружен на текущем execution path, подумай, можно ли безопасно переиспользовать его или вычислить capabilities одним вызовом.

---

# 7. Validation / DTO / runtime contract

Проверь соответствие:

- TypeScript type;
- runtime validator;
- external payload;
- persisted representation.

Особенно:

- `field?: string` vs `field: string | null`;
- `null` vs `undefined`;
- empty string;
- whitespace;
- enum/string;
- array vs nullable array;
- transformed query/path/body values.

### Правило canonical contract

Если parser/DTO/boundary уже гарантирует canonical representation, downstream NestJS service не должен снова строить другой контракт без причины.

Если разные consumers одного DTO трактуют поле по-разному — это finding.

---

# 8. ConfigService / env

Для каждой новой config-переменной проверь:

- `get` или `getOrThrow`;
- default;
- Joi/Zod/env validation, если проект её использует;
- `.env` / `.env.example` / Docker env;
- compose/deploy config;
- тесты;
- документацию.

Если от значения зависят:

- admin identity;
- authorization;
- destructive cleanup;
- внешняя интеграция;
- startup correctness;

silent placeholder default обычно опаснее fail-fast.

Не считать наличие default достаточным доказательством корректной конфигурации.

---

# 9. Exceptions и logging

Проверь:

- `HttpException` действительно уместен на этом слое;
- несколько веток не бросают одинаковый bare exception без диагностического контекста;
- handler/logging не теряет root cause;
- catch не классифицирует чужую ошибку как ошибку внешнего подключения;
- ошибка formatter/notification не запускает ненужный retry sync-процесса;
- logging не дублируется на каждом слое;
- credentials/tokens не попадают в logs.

Если общий handler пишет `error.message`, сообщения исключений должны помогать понять, какая ветка сработала.

---

# 10. Workers / Cron / Schedulers

Особенно проверяй:

- overlapping execution;
- duplicate processing;
- idempotency;
- retry;
- lock strategy;
- shutdown;
- partial side effects;
- повторный запуск;
- stale state;
- cron, который может отменить результат manual action;
- I/O внутри вложенных циклов;
- repeated DB reads;
- write amplification.

Для каждого DB write в периодическом процессе спроси:

> Это изменение реально произошло или мы переписываем ту же строку каждый проход?

Для каждого DB read внутри цикла спроси:

> Можно ли выполнить один запрос на entity/pass вместо occurrence × connection?

Порядок дешёвых проверок и I/O тоже является частью review.

---

# 11. Sequelize / TypeORM transactions

Проверь:

- корректную transaction boundary;
- одинаковый стиль optional transaction;
- транзакция передаётся во все связанные writes;
- external call не удерживает DB transaction без необходимости;
- lock semantics видимы;
- обычный `find*` не превращается неочевидно в `FOR UPDATE`;
- rollback не оставляет внешний side effect в полусостоянии;
- N последовательных update не появились случайно вместо существующего bulk operation.

Если blocking read нужен по смыслу, предпочитай явно именованный API, например `find...ForUpdate`, когда это соответствует conventions проекта.

Потенциально длинная транзакция без доказанного дефекта может быть `PERFORMANCE_DEFECT`/`SPECULATIVE`, а не обязательным fix.

---

# 12. Repository access patterns

Ищи:

- два одинаковых запроса в одной транзакции;
- широкую выборку с дальнейшей фильтрацией по нескольким ID/href;
- query per user/connection/occurrence;
- повторное чтение одного User для guard + handler;
- UPDATE без проверки, изменилось ли значение;
- запросы до `dedup`/cheap guard;
- repository API, который скрывает важные side effects.

Не требуй caching автоматически. Учитывай freshness contract.

---

# 13. NestJS background failure radius

Для worker/cron/service проверь место `try/catch`.

Спроси:

- что именно retried при исключении;
- не окажется ли ошибка formatting/send внутри catch блока "connection sync failed";
- не повторит ли retry уже совершённые side effects;
- не упадёт ли весь daily cron из-за одной плохой записи;
- должен ли один invalid external record ломать весь batch.

Catch boundary должна совпадать с единицей работы и retry semantics.

---

# 14. Tests: NestJS-specific integrity

Помимо общего mutation review из `senior-review`, проверь:

- unit не поднимает весь Nest TestingModule без необходимости;
- integration действительно проверяет wiring, если wiring является риском;
- e2e не мокает production provider, который и должен быть проверен;
- overrideProvider не скрывает нужную интеграцию;
- mocked provider не воспроизводит production implementation;
- lifecycle hooks реально запускаются там, где это важно;
- test helper не добавляет опасные default arguments;
- `toHaveBeenCalledWith` не матчит более ранний вызов service/repository;
- exception assertion проверяет нужную ветку, а не любой reject.

Если название теста обещает проверку NestJS wiring, но provider вручную создан через `new`, такой тест не доказывает wiring.

---

# 15. Shared e2e application state

Если e2e suite использует общий Nest application / DB:

- test не должен сносить общую таблицу без ownership-фильтра;
- migration test должен восстановить схему;
- cleanup должен удалять только собственные records;
- test order не должен менять результат;
- `afterAll` должен восстанавливать глобальные overrides/state;
- cron/timers/listeners должны закрываться.

Считай тест подозрительным, если он зелёный только когда запускается последним.

---

# 16. Templates / renderers / user-visible output

Для NestJS feature, которая формирует сообщения/шаблоны:

- сравни условия показа до/после;
- проверь, что удаление role/guard не сделало блок видимым всем;
- сравни worker/command/notification paths;
- проверь timezone и all-day semantics;
- проверь старые persisted `''`/null значения.

Механическая замена условия в template является behavioral change, даже если TypeScript-код почти не изменился.

---

# 17. Deletion cleanup

Если удаляется module/integration/provider:

проверь также:

- module imports;
- exports;
- npm/yarn dependency;
- env config;
- Docker env;
- credentials;
- docs;
- tests;
- injection tokens;
- dead interfaces/types.

Особенно не оставляй секреты конфигурации удалённой интеграции.

---

# 18. Формат finding

Используй общий формат `senior-review`/`code-review`.

Для NestJS-specific finding желательно указывать:

- affected module/provider/worker;
- runtime execution path;
- почему TypeScript/DI/tests это не ловят;
- каков failure radius;
- fix-now это или отдельная задача.

Если NestJS-специфичных проблем нет:

`NESTJS REVIEW: CLEAN`

Не возвращай CLEAN, если wiring/runtime path нельзя было проверить из доступного контекста.

---

# 19. Anti-noise

Не выдавай generic NestJS best practices без связи с изменением.

Не требуй:

- `forwardRef` убрать "потому что плохо";
- controller сделать тоньше без реального эффекта;
- отдельный abstraction "на будущее";
- repository слой переписать ради вкуса;
- request scope использовать без конкретной причины.

Нужен доказуемый defect/risk, а не архитектурная религия.

---

# 20. Language

All user-facing output must be in Russian.
