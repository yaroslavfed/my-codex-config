# Маршруты чата

## Текущее поведение

- `src/services/chat-server/chat.routes.ts` вызывает gRPC-клиент [[elph-chat-server]]
- Отправка сообщения: `POST /v1/chat.sendMessage`, тело содержит `message`. Роут передаёт его через `toNewMessage` в gRPC `ChatService.SendMessage` и возвращает `message` с `success: true`
- Inline-команда: `POST /v1/chat.runInlineCommand` с `rid`, `sourceMessageId`, `commandId`
- Дельта обновлений: `GET /v1/chat.updatedMessages` с `roomId`, `updatedSince` и параметрами страницы
- Регистрация chat-маршрутов применяет обязательную HTTP-аутентификацию и feature gate `chats`; редактирование, удаление и другие отдельные операции имеют дополнительные gates

## Источники

- `src/services/chat-server/chat.routes.ts`, `registerChatRoutes`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/services/chat-server/register.ts`, `registerChatServer`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
