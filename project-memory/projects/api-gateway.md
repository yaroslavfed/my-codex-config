# api-gateway

## Назначение

Внешний HTTP-вход для chat-сценариев. Маршруты в `src/services/chat-server/chat.routes.ts` вызывают gRPC-клиент chat-server.

## Устройство

- `TokenVerifier` направляет бот-токены с issuer `elph-chat-bot` на gRPC `ChatBotService.VerifyBotToken` chat-server и кэширует проверенную личность
- DDP `login` проверяет переданный `resume` через `TokenVerifier`; HTTP `/elph_chat/api/v1/login` передаёт параметры в `AuthService.Login` chat-server

## Контракты и сценарии

- Отправка сообщения: `POST /v1/chat.sendMessage`, тело содержит `message`. Роут передаёт его через `toNewMessage` в gRPC `ChatService.SendMessage` и возвращает `message` с `success: true`
- Inline-команда: `POST /v1/chat.runInlineCommand` с `rid`, `sourceMessageId`, `commandId`
- Дельта обновлений: `GET /v1/chat.updatedMessages` с `roomId`, `updatedSince` и параметрами страницы
- Регистрация chat-маршрутов применяет обязательную HTTP-аутентификацию и feature gate `chats`; редактирование, удаление и другие отдельные операции имеют дополнительные gates
- Административные маршруты `/elph_chat/api/v1/bots.create`, `bots.update`, `bots.regenerate-token` обращаются к chat-server; `bots.register-subscription` обращается к bot-service. Все четыре доступны отдельно от сценариев Workshop
- `/bot-api/v1` обслуживает запросы уже созданного бота, включая сообщения, webhook и клавиатуры; его вызывает бот с собственным токеном

## Особенности и риски

- При отзыве бот-токена проверенная личность может оставаться в кэше gateway до истечения настроенного TTL

## Связи

- [[elph-chat-server]] — gRPC upstream для chat-маршрутов, включая отправку сообщения, получение обновлений и текущую проверку бот-токена
- [[elph-bot-service]] — gRPC upstream для Bot API и ручной регистрации подписки

## Источники

- `src/services/chat-server/chat.routes.ts`, `registerChatRoutes`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/services/chat-server/register.ts`, `registerChatServer`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/services/chat-server/bots.routes.ts`, `registerBotsRoutes`; `src/services/bot-api/register.ts`, `registerBotApi`, проверено по `origin/develop/3.9.0` 2026-11-09
- `src/platform/auth/verifier.ts`, `TokenVerifier`; `src/services/chat-server/ddp/methods.ts`, `chatDdpMethods`; `src/services/chat-server/login.routes.ts`, `registerChatLoginRoute`, проверено по `origin/develop/3.9.0` 2026-11-09
