# Сообщения

## Текущее поведение

- `ChatService` и модель `Messages` реализуют правила и выборки сообщений
- `ChatGrpcController.sendMessage` проверяет право отправки, вызывает `ChatService.sendMessage` и нормализует ответ для клиентов
- gRPC `ChatService.SendMessage` — отправка сообщения. Внешняя HTTP-ручка в [[api-gateway/chat-routes|api-gateway]]: `POST /v1/chat.sendMessage`
- `ChatService.getUpdatedMessages` проверяет доступ к комнате, читает обновлённые сообщения и нормализует их для пользователя
- gRPC `ChatService.UpdatedMessages` — дельта обновлений. Внешняя HTTP-ручка: `GET /v1/chat.updatedMessages`
- `MessageEventGenerator` формирует события сообщений для брокера и ботов

## Особенности и риски

- Подтверждённый по коду риск: `Messages.findUpdatedInRoomTree` не фильтрует `_hidden`, а `ChatService.getUpdatedMessages` передаёт найденные сообщения в `normalizeForUser`. Требование скрывать такие записи на этом пути нельзя считать выполненным без исправления и проверки полного пути

## Источники

- `src/grpc/chat.grpc.controller.ts`, `sendMessage`, `updatedMessages`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/api/chat/chat.service.ts`, `getUpdatedMessages`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/database/models/raw/Messages.ts`, `findUpdatedInRoomTree`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/lib/event-generators/message-event-generator.ts`, `MessageEventGenerator`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
