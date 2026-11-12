# Inline-команды

## Текущее поведение

- gRPC `ChatService.RunInlineCommand` — исполнение inline-команды; внешний HTTP-вход: `POST /v1/chat.runInlineCommand` в [[api-gateway/chat-routes|api-gateway]]

## Особенности и риски

- Событие inline-команды боту публикуется через `publishAndWait`, в отличие от обычного пакетного пути. Не полагайся на его порядок относительно обычных событий сообщений

## Источники

- `src/grpc/chat.grpc.controller.ts`, `runInlineCommand`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/lib/event-generators/message-event-generator.ts`, `deliverInlineCommand`, `deliverMessageEvent`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
