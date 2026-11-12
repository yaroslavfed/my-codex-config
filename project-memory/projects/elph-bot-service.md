# elph-bot-service

## Назначение

Принимает события чата для управляемых ботов и направляет их в обработчики согласно виду бота и получателю события. Управляет созданием, ротацией и отключением ботов через сценарии Workshop.

## Устройство

- `BotSubscriptionRegistry` восстанавливает подписки по списку разрешённых ботов из profile-service при запуске и передаёт сообщения зарегистрированному обработчику
- `BotIngressProcessor` декодирует событие и определяет получателя
- `BotMessageListener` направляет `custom` в очередь webhook, `workshop` в state machine и `conference` в свой обработчик; для Workshop и Conference допускает только прямые сообщения с получателем `message`
- В ветке `refs_550675` `CreateBotUseCase`, `RevokeBotUseCase` и `DisableBotUseCase` работают с токеном через локальный `BotTokensService` и PostgreSQL; личность бота остаётся в profile-service
- Схема PostgreSQL версионируется через `database_info`; миграции выполняются при запуске под advisory lock, версия 2 однократно импортирует токены из chat-server

## Контракты и сценарии

- `BotService.VerifyBotToken` сверяет точную строку токена с PostgreSQL и запрашивает актуальную личность бота у profile-service
- gRPC `BotService.RegisterBotSubscription` вручную регистрирует действующего бота после проверки в profile-service

## Особенности и риски

- Старое деление работ на «task 3» и «task 5» описывало план внедрения и не является текущей схемой сервиса. В проверенной ветке входящие подписки и маршрутизация в Workshop уже присутствуют
- Сервис использует Redis для подписок; в ветке `refs_550675` бот-токены хранятся в собственной PostgreSQL, импортированные строки продолжают проверяться без перевыпуска
- Импорт токенов из chat-server однократный по версии схемы, а деплой идёт RollingUpdate: токены, выданные старыми подами в chat после миграции нового пода, в PostgreSQL не попадут и перестанут проходить проверку в gateway

## Связи

- [[elph-chat-server]] — источник событий сообщений и разовой выгрузки ранее выданных бот-токенов
- [[api-gateway]] — вызывает сервис для Bot API и ручной регистрации подписки

## Источники

- `src/bot-registry/bot-subscription-registry.ts`, `BotSubscriptionRegistry`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/bot-ingress/bot-ingress.processor.ts`, `BotIngressProcessor`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/bot-ingress/bot-message.listener.ts`, `BotMessageListener`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/lifecycle/create-bot.use-case.ts`, `CreateBotUseCase`; `src/lifecycle/revoke-bot.use-case.ts`, `RevokeBotUseCase`; `src/lifecycle/disable-bot.use-case.ts`, `DisableBotUseCase`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/integrations/chat/chat-bot-token.adapter.ts`, `ChatBotTokenAdapter`; `src/bot-registry/bot-subscription-registry.ts`, `BotSubscriptionRegistry`; `src/bot-registry/register-bot-subscription.use-case.ts`, `RegisterBotSubscriptionUseCase`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/bot-tokens/bot-tokens.service.ts`, `BotTokensService`; `src/database/database-migrations.service.ts`, `DatabaseMigrationsService`; `src/database/v002-import-chat-bot-tokens.ts`; `src/integrations/chat/chat-bot-tokens-export.adapter.ts`, проверено по `refs_550675` 2026-11-12
