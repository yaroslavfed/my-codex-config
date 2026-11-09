# platform-contracts

## Назначение

Содержит gRPC-контракты сервисов и общие типы платформы.

## Контракты и сценарии

- `elph.chat.v1.ChatBotService` в `internal.proto` описывает служебную проверку, выдачу, ротацию и удаление бот-токена
- `elph.chat.v1.BotsService` описывает создание, правку бота и перевыпуск токена для административных клиентских маршрутов
- `elph.bot.v1.BotService` описывает Bot API и ручную регистрацию подписки бота

## Связи

- [[elph-chat-server]] — реализует `ChatBotService` и `BotsService`
- [[elph-bot-service]] — реализует `elph.bot.v1.BotService` и сейчас вызывает токенные методы чата
- [[api-gateway]] — вызывает все три gRPC-сервиса через сгенерированных клиентов

## Источники

- `proto/elph/chat/v1/internal.proto`, `ChatBotService`; `proto/elph/chat/v1/bots.proto`, `BotsService`; `proto/elph/bot/v1/bot.proto`, `BotService`, проверено по `origin/develop/3.9.0` 2026-11-09
