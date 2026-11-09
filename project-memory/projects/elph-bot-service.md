# elph-bot-service

## Назначение

Принимает события чата для управляемых ботов и направляет их в обработчики согласно виду бота и получателю события. Управляет созданием, ротацией и отключением ботов через сценарии Workshop.

## Устройство

- `BotSubscriptionRegistry` восстанавливает подписки по списку разрешённых ботов из profile-service при запуске и передаёт сообщения зарегистрированному обработчику
- `BotIngressProcessor` декодирует событие и определяет получателя
- `BotMessageListener` направляет `custom` в очередь webhook, `workshop` в state machine и `conference` в свой обработчик; для Workshop и Conference допускает только прямые сообщения с получателем `message`
- `CreateBotUseCase`, `RevokeBotUseCase` и `DisableBotUseCase` работают с личностью бота через profile-service, а с токеном через `ChatBotTokenAdapter` к chat-server

## Контракты и сценарии

- `ChatBotTokenAdapter` вызывает `IssueBotToken`, `RotateBotToken` и `DeleteBotToken` в `ChatBotService` chat-server; клиентская проверка токена через этот адаптер не проходит
- gRPC `BotService.RegisterBotSubscription` вручную регистрирует действующего бота после проверки в profile-service

## Особенности и риски

- Старое деление работ на «task 3» и «task 5» описывало план внедрения и не является текущей схемой сервиса. В проверенной ветке входящие подписки и маршрутизация в Workshop уже присутствуют
- Сервис использует Redis для подписок, но в проверенном коде не хранит в нём бот-токены. Источник токенов сейчас находится в chat-server

## Связи

- [[elph-chat-server]] — источник событий сообщений и текущий поставщик операций с бот-токенами
- [[api-gateway]] — вызывает сервис для Bot API и ручной регистрации подписки

## Источники

- `src/bot-registry/bot-subscription-registry.ts`, `BotSubscriptionRegistry`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/bot-ingress/bot-ingress.processor.ts`, `BotIngressProcessor`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/bot-ingress/bot-message.listener.ts`, `BotMessageListener`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/lifecycle/create-bot.use-case.ts`, `CreateBotUseCase`; `src/lifecycle/revoke-bot.use-case.ts`, `RevokeBotUseCase`; `src/lifecycle/disable-bot.use-case.ts`, `DisableBotUseCase`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/integrations/chat/chat-bot-token.adapter.ts`, `ChatBotTokenAdapter`; `src/bot-registry/bot-subscription-registry.ts`, `BotSubscriptionRegistry`; `src/bot-registry/register-bot-subscription.use-case.ts`, `RegisterBotSubscriptionUseCase`, проверено по `origin/develop/3.9.0` 2026-11-09
