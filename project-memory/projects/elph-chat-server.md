# elph-chat-server

## Назначение

Сервис хранения и обработки чата. gRPC-контроллер предоставляет операции над сообщениями; `ChatService` и модель `Messages` реализуют связанные правила и выборки.

## Устройство

- `ChatGrpcController.sendMessage` проверяет право отправки, вызывает `ChatService.sendMessage` и нормализует ответ для клиентов
- `MessageEventGenerator` формирует события сообщений для брокера и ботов
- `ChatService.getUpdatedMessages` проверяет доступ к комнате, читает обновлённые сообщения и нормализует их для пользователя
- `BotTokenService` выпускает, ротирует, удаляет и проверяет бот-токены через MongoDB `auth_services`; `AuthModule` регистрирует его в общей аутентификации

## Контракты и сценарии

- gRPC `ChatService.SendMessage` — отправка сообщения. Внешняя HTTP-ручка расположена в [[api-gateway]]: `POST /v1/chat.sendMessage`
- gRPC `ChatService.RunInlineCommand` — исполнение inline-команды; внешний HTTP-вход: `POST /v1/chat.runInlineCommand`
- gRPC `ChatService.UpdatedMessages` — дельта обновлений; внешний HTTP-вход: `GET /v1/chat.updatedMessages`
- В `ChatBotService` сохранены служебные `VerifyBotToken`, `IssueBotToken`, `RotateBotToken`, `DeleteBotToken`; `BotsService` сохраняет `Create`, `Update`, `RegenerateToken`, хотя прямые маршруты gateway к ним в ветке `refs_550675` удалены
- В ветке `refs_550675` добавлен `ChatBotService.ListBotTokens`: он читает `auth_services` с `service: 'bot-jwt'`, отдаёт текущий токен и последний идентификатор операции, повреждённые записи пропускает и считает в журнале без вывода токенов

## Особенности и риски

- Событие inline-команды боту публикуется через `publishAndWait`, в отличие от обычного пакетного пути. Не полагайся на его порядок относительно обычных событий сообщений
- Подтверждённый по коду риск: `Messages.findUpdatedInRoomTree` не фильтрует `_hidden`, а `ChatService.getUpdatedMessages` передаёт найденные сообщения в `normalizeForUser`. Требование скрывать такие записи на этом пути нельзя считать выполненным без исправления и проверки полного пути
- Бот-токен хранится в `auth_services` как точная строка; `BotTokenService.verify` декодирует JWT и сравнивает строку с хранилищем. Выдача идемпотентна, ротация использует `operationId` и условное обновление записи
- `ResumeAuthService` отдельно пропускает `bot-jwt` без перевыпуска, поэтому удаление бот-токенов из общей аутентификации затрагивает сценарий `resume`

## Связи

- [[api-gateway]] — публичные HTTP-маршруты преобразуются в gRPC-вызовы этого сервиса
- [[elph-bot-service]] — получает события сообщений для маршрутизации управляемым ботам и однократно выгружает токены через `ListBotTokens` при миграции PostgreSQL; прежние токенные RPC чата пока сохранены

## Источники

- `src/grpc/chat.grpc.controller.ts`, `sendMessage`, `runInlineCommand`, `updatedMessages`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/api/chat/chat.service.ts`, `getUpdatedMessages`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/database/models/raw/Messages.ts`, `findUpdatedInRoomTree`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/lib/event-generators/message-event-generator.ts`, `deliverInlineCommand`, `deliverMessageEvent`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/api/authentication/token/BotTokenService.ts`, `BotTokenService`; `src/api/authentication/auth.module.ts`, `AuthModule`; `src/api/authentication/resume/ResumeAuthService.ts`, `ResumeAuthService`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/grpc/internal.grpc.controller.ts`, `InternalGrpcController`; `src/grpc/bots.grpc.controller.ts`, `BotsGrpcController`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/api/authentication/token/BotTokenService.ts`, `list`; `src/grpc/internal.grpc.controller.ts`, `listBotTokens`, проверено по `refs_550675` 2026-11-12
