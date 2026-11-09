# elph-bot-service

## Назначение

Принимает события чата для управляемых ботов и направляет их в обработчики согласно виду бота и получателю события.

## Устройство

- `BotSubscriptionRegistry` восстанавливает подписки на разрешённых ботов при запуске и передаёт сообщения зарегистрированному обработчику
- `BotIngressProcessor` декодирует событие и определяет получателя
- `BotMessageListener` направляет `custom` в очередь webhook, `workshop` в state machine и `conference` в свой обработчик; для Workshop и Conference допускает только прямые сообщения с получателем `message`

## Особенности и риски

- Старое деление работ на «task 3» и «task 5» описывало план внедрения и не является текущей схемой сервиса. В проверенной ветке входящие подписки и маршрутизация в Workshop уже присутствуют

## Связи

- [[elph-chat-server]] — источник событий сообщений, которые принимает ingress ботов

## Источники

- `src/bot-registry/bot-subscription-registry.ts`, `BotSubscriptionRegistry`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/bot-ingress/bot-ingress.processor.ts`, `BotIngressProcessor`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
- `src/bot-ingress/bot-message.listener.ts`, `BotMessageListener`, проверено в локальной ветке `develop/3.9.0` 2026-11-09
