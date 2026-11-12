# platform-contracts

## Назначение

Содержит gRPC-контракты сервисов и общие типы платформы.

## Контракты и сценарии

- `elph.chat.v1.ChatBotService` в `internal.proto` сохраняет прежние токенные RPC и предоставляет `ListBotTokens` для разовой выгрузки токенов с последним `operation_id`
- `elph.chat.v1.BotsService` сохраняет контракты создания, правки бота и перевыпуска токена; прямые клиентские маршруты gateway в ветке `refs_550675` удалены
- `elph.bot.v1.BotService` описывает Bot API, регистрацию подписки и `VerifyBotToken` с полями личности, совместимыми с прежним ответом чата

## Связи

- [[elph-chat-server]] — реализует `ChatBotService` и `BotsService`
- [[elph-bot-service]] — реализует `elph.bot.v1.BotService` и вызывает `ChatBotService.ListBotTokens` при первом запуске с новой схемой
- [[api-gateway]] — вызывает `BotService.VerifyBotToken` для проверки бот-токенов

## Источники

- `proto/elph/chat/v1/internal.proto`, `ChatBotService`; `proto/elph/chat/v1/bots.proto`, `BotsService`; `proto/elph/bot/v1/bot.proto`, `BotService`, проверено по `origin/develop/3.9.0` 2026-11-09
- `proto/elph/chat/v1/internal.proto`, `ListBotTokens`; `proto/elph/bot/v1/bot.proto`, `VerifyBotToken`, проверено по `refs_550675` 2026-11-12
