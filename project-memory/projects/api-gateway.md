# api-gateway

## Назначение

Внешний HTTP-вход для chat-сценариев. Маршруты в `src/services/chat-server/chat.routes.ts` вызывают gRPC-клиент chat-server.

## Устройство

- В ветке `refs_550675` `TokenVerifier` направляет бот-токены с issuer `elph-chat-bot` на gRPC `BotService.VerifyBotToken` bot-service и кэширует проверенную личность
- DDP `login` проверяет переданный `resume` через `TokenVerifier`; HTTP `/elph_chat/api/v1/login` передаёт параметры в `AuthService.Login` chat-server

## Контракты и сценарии

- Отправка сообщения: `POST /v1/chat.sendMessage`, тело содержит `message`. Роут передаёт его через `toNewMessage` в gRPC `ChatService.SendMessage` и возвращает `message` с `success: true`
- Inline-команда: `POST /v1/chat.runInlineCommand` с `rid`, `sourceMessageId`, `commandId`
- Дельта обновлений: `GET /v1/chat.updatedMessages` с `roomId`, `updatedSince` и параметрами страницы
- Регистрация chat-маршрутов применяет обязательную HTTP-аутентификацию и feature gate `chats`; редактирование, удаление и другие отдельные операции имеют дополнительные gates
- В ветке `refs_550675` административные маршруты `/elph_chat/api/v1/bots.create`, `bots.update` и `bots.regenerate-token` удалены, а `bots.register-subscription` с проверкой роли admin сохранён до очистки токенов в chat-server; управление ботами остаётся в сценариях Workshop
- `/bot-api/v1` обслуживает запросы уже созданного бота, включая сообщения, webhook и клавиатуры; его вызывает бот с собственным токеном
- Действующие `aggregateElphUserStatuses(userIds)` и `elphUserStatusChanged(userIds)` получают снимок и события из `elph-user-status`. По проекту рабочих статусов от 2026-11-12 эти же операции должны вернуть третье nullable поле `workStatus`; в текущих GraphQL SDL и мапперах поля пока нет.

## Особенности и риски

- При отзыве бот-токена проверенная личность может оставаться в кэше gateway до истечения настроенного TTL

## Связи

- [[elph-chat-server]] — gRPC upstream для chat-маршрутов, включая отправку сообщения и получение обновлений
- [[elph-bot-service]] — gRPC upstream для Bot API и проверки бот-токенов
- [[elph-user-status]] — gRPC upstream для снимка статуса и источник событий для GraphQL-подписки

## Источники

- `src/services/chat-server/chat.routes.ts`, `registerChatRoutes`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/services/chat-server/register.ts`, `registerChatServer`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/services/chat-server/bots.routes.ts`, `registerBotsRoutes`; `src/services/bot-api/register.ts`, `registerBotApi`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/platform/auth/verifier.ts`, `TokenVerifier`; `src/services/chat-server/ddp/methods.ts`, `chatDdpMethods`; `src/services/chat-server/login.routes.ts`, `registerChatLoginRoute`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/platform/auth/verifier.ts`, `TokenVerifier`; `src/app.ts`, `buildApp`; `src/services/chat-server/register.ts`, `registerChatServer`, проверено по `refs_550675` 2026-11-12
- `src/services/user-status/graphql/schema.ts`, `resolvers.ts`, `mappers.ts`, `aggregateElphUserStatuses`, `elphUserStatusChanged`, локальная ветка `refs_550675`, проверено 2026-11-12; символы найдены WebStorm index
- Целевой контракт: `/home/iaroslavfed/Downloads/Клиентский контракт рабочих статусов УРВ-v11-20261112_142913.pdf`, редакция 2026-11-12
