# api-gateway

## Назначение

Внешний HTTP-вход для chat-сценариев. Маршруты в `src/services/chat-server/chat.routes.ts` вызывают gRPC-клиент chat-server.

## Контракты и сценарии

- Отправка сообщения: `POST /v1/chat.sendMessage`, тело содержит `message`. Роут передаёт его через `toNewMessage` в gRPC `ChatService.SendMessage` и возвращает `message` с `success: true`
- Inline-команда: `POST /v1/chat.runInlineCommand` с `rid`, `sourceMessageId`, `commandId`
- Дельта обновлений: `GET /v1/chat.updatedMessages` с `roomId`, `updatedSince` и параметрами страницы
- Регистрация chat-маршрутов применяет обязательную HTTP-аутентификацию и feature gate `chats`; редактирование, удаление и другие отдельные операции имеют дополнительные gates

## Связи

- [[elph-chat-server]] — gRPC upstream для chat-маршрутов, включая отправку сообщения и получение обновлений

## Источники

- `src/services/chat-server/chat.routes.ts`, `registerChatRoutes`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/services/chat-server/register.ts`, `registerChatServer`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
