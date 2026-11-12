# Маршрутизация событий

## Текущее поведение

- `BotSubscriptionRegistry` восстанавливает подписки по списку разрешённых ботов из profile-service при запуске и передаёт сообщения зарегистрированному обработчику
- `BotIngressProcessor` декодирует событие и определяет получателя
- `BotMessageListener` направляет `custom` в очередь webhook, `workshop` в state machine и `conference` в свой обработчик; для Workshop и Conference допускает только прямые сообщения с получателем `message`
- gRPC `BotService.RegisterBotSubscription` вручную регистрирует действующего бота после проверки в profile-service
- Сервис использует Redis для подписок

## Особенности

- Старое деление работ на «task 3» и «task 5» описывало план внедрения и не является текущей схемой сервиса. В проверенной ветке входящие подписки и маршрутизация в Workshop уже присутствуют

## Связи

- [[elph-chat-server/messages|elph-chat-server]] — публикует события сообщений
- [[api-gateway/bot-auth|api-gateway]] — вызывает ручную регистрацию подписки

## Источники

- `src/bot-registry/bot-subscription-registry.ts`, `BotSubscriptionRegistry`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/bot-ingress/bot-ingress.processor.ts`, `BotIngressProcessor`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/bot-ingress/bot-message.listener.ts`, `BotMessageListener`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/integrations/chat/chat-bot-token.adapter.ts`, `ChatBotTokenAdapter`; `src/bot-registry/bot-subscription-registry.ts`, `BotSubscriptionRegistry`; `src/bot-registry/register-bot-subscription.use-case.ts`, `RegisterBotSubscriptionUseCase`, проверено по `origin/develop/3.9.0` 2026-11-09
