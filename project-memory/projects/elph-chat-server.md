# elph-chat-server

## Назначение

Сервис хранения и обработки чата. gRPC-контроллер предоставляет операции над сообщениями; `ChatService` и модель `Messages` реализуют связанные правила и выборки.

## Устройство

- `ChatGrpcController.sendMessage` проверяет право отправки, вызывает `ChatService.sendMessage` и нормализует ответ для клиентов
- `MessageEventGenerator` формирует события сообщений для брокера и ботов
- `ChatService.getUpdatedMessages` проверяет доступ к комнате, читает обновлённые сообщения и нормализует их для пользователя

## Контракты и сценарии

- gRPC `ChatService.SendMessage` — отправка сообщения. Внешняя HTTP-ручка расположена в [[api-gateway]]: `POST /v1/chat.sendMessage`
- gRPC `ChatService.RunInlineCommand` — исполнение inline-команды; внешний HTTP-вход: `POST /v1/chat.runInlineCommand`
- gRPC `ChatService.UpdatedMessages` — дельта обновлений; внешний HTTP-вход: `GET /v1/chat.updatedMessages`

## Особенности и риски

- Событие inline-команды боту публикуется через `publishAndWait`, в отличие от обычного пакетного пути. Не полагайся на его порядок относительно обычных событий сообщений
- Подтверждённый по коду риск: `Messages.findUpdatedInRoomTree` не фильтрует `_hidden`, а `ChatService.getUpdatedMessages` передаёт найденные сообщения в `normalizeForUser`. Требование скрывать такие записи на этом пути нельзя считать выполненным без исправления и проверки полного пути

## Связи

- [[api-gateway]] — публичные HTTP-маршруты преобразуются в gRPC-вызовы этого сервиса
- [[elph-bot-service]] — получает события сообщений для маршрутизации управляемым ботам

## Источники

- `src/grpc/chat.grpc.controller.ts`, `sendMessage`, `runInlineCommand`, `updatedMessages`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/api/chat/chat.service.ts`, `getUpdatedMessages`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/database/models/raw/Messages.ts`, `findUpdatedInRoomTree`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/lib/event-generators/message-event-generator.ts`, `deliverInlineCommand`, `deliverMessageEvent`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
