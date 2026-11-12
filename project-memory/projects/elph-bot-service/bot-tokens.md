# Бот-токены

## Проверенное поведение

- В ветке `refs_550675` `CreateBotUseCase`, `RevokeBotUseCase` и `DisableBotUseCase` работают с токеном через локальный `BotTokensService` и PostgreSQL; личность бота остаётся в profile-service
- Схема PostgreSQL версионируется через `database_info`; миграции выполняются при запуске под advisory lock, версия 2 однократно импортирует токены из [[elph-chat-server/bot-tokens|elph-chat-server]]
- В ветке `refs_550675` gRPC `BotService.VerifyBotToken` сверяет точную строку токена с PostgreSQL и запрашивает актуальную личность бота у profile-service
- В ветке `refs_550675` бот-токены хранятся в собственной PostgreSQL, импортированные строки продолжают проверяться без перевыпуска

## Особенности и риски

- Импорт токенов из chat-server однократный по версии схемы, а деплой идёт RollingUpdate: токены, выданные старыми подами в chat после миграции нового пода, в PostgreSQL не попадут и перестанут проходить проверку в gateway

## Связи

- [[api-gateway/bot-auth|api-gateway]] — направляет проверку бот-токенов в этот сервис
- [[helm-charts-elph]] — настройки PostgreSQL для сервиса

## Источники

- `src/lifecycle/create-bot.use-case.ts`, `CreateBotUseCase`; `src/lifecycle/revoke-bot.use-case.ts`, `RevokeBotUseCase`; `src/lifecycle/disable-bot.use-case.ts`, `DisableBotUseCase`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/bot-tokens/bot-tokens.service.ts`, `BotTokensService`; `src/database/database-migrations.service.ts`, `DatabaseMigrationsService`; `src/database/v002-import-chat-bot-tokens.ts`; `src/integrations/chat/chat-bot-tokens-export.adapter.ts`, проверено по `refs_550675` 2026-11-12
