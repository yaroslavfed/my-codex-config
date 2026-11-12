# Бот-токены

## Проверенное поведение

- `BotTokenService` выпускает, ротирует, удаляет и проверяет бот-токены через MongoDB `auth_services`; `AuthModule` регистрирует его в общей аутентификации
- В `ChatBotService` сохранены служебные `VerifyBotToken`, `IssueBotToken`, `RotateBotToken`, `DeleteBotToken`; `BotsService` сохраняет `Create`, `Update`, `RegenerateToken`, хотя прямые маршруты gateway к ним в ветке `refs_550675` удалены
- В ветке `refs_550675` добавлен `ChatBotService.ListBotTokens`: он читает `auth_services` с `service: 'bot-jwt'`, отдаёт текущий токен и последний идентификатор операции, повреждённые записи пропускает и считает в журнале без вывода токенов

## Особенности и риски

- Бот-токен хранится в `auth_services` как точная строка; `BotTokenService.verify` декодирует JWT и сравнивает строку с хранилищем. Выдача идемпотентна, ротация использует `operationId` и условное обновление записи
- `ResumeAuthService` отдельно пропускает `bot-jwt` без перевыпуска, поэтому удаление бот-токенов из общей аутентификации затрагивает сценарий `resume`

## Связи

- [[elph-bot-service/bot-tokens|elph-bot-service]] — однократно выгружает токены через `ListBotTokens` при миграции PostgreSQL; прежние токенные RPC чата пока сохранены

## Источники

- `src/api/authentication/token/BotTokenService.ts`, `BotTokenService`; `src/api/authentication/auth.module.ts`, `AuthModule`; `src/api/authentication/resume/ResumeAuthService.ts`, `ResumeAuthService`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/grpc/internal.grpc.controller.ts`, `InternalGrpcController`; `src/grpc/bots.grpc.controller.ts`, `BotsGrpcController`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/api/authentication/token/BotTokenService.ts`, `list`; `src/grpc/internal.grpc.controller.ts`, `listBotTokens`, проверено по `refs_550675` 2026-11-12
