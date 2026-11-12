# Боты и аутентификация

## Проверенное поведение

- В ветке `refs_550675` `TokenVerifier` направляет бот-токены с issuer `elph-chat-bot` на gRPC `BotService.VerifyBotToken` [[elph-bot-service]] и кэширует проверенную личность
- DDP `login` проверяет переданный `resume` через `TokenVerifier`; HTTP `/elph_chat/api/v1/login` передаёт параметры в `AuthService.Login` [[elph-chat-server]]
- В ветке `refs_550675` административные маршруты `/elph_chat/api/v1/bots.create`, `bots.update` и `bots.regenerate-token` удалены, а `bots.register-subscription` с проверкой роли admin сохранён до очистки токенов в chat-server; управление ботами остаётся в сценариях Workshop
- `/bot-api/v1` обслуживает запросы уже созданного бота, включая сообщения, webhook и клавиатуры; его вызывает бот с собственным токеном

## Особенности и риски

- При отзыве бот-токена проверенная личность может оставаться в кэше gateway до истечения настроенного TTL

## Источники

- `src/services/chat-server/bots.routes.ts`, `registerBotsRoutes`; `src/services/bot-api/register.ts`, `registerBotApi`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/platform/auth/verifier.ts`, `TokenVerifier`; `src/services/chat-server/ddp/methods.ts`, `chatDdpMethods`; `src/services/chat-server/login.routes.ts`, `registerChatLoginRoute`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/platform/auth/verifier.ts`, `TokenVerifier`; `src/app.ts`, `buildApp`; `src/services/chat-server/register.ts`, `registerChatServer`, проверено по `refs_550675` 2026-11-12
