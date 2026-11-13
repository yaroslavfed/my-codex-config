# api-gateway

Внешний HTTP-вход для сценариев чата, ботов и статусов пользователей

## Архитектура

[[api-gateway.canvas|Схема API Gateway]]

## Темы

- [[api-gateway/chat-routes|Маршруты чата]] — отправка сообщений, inline-команды и получение обновлений
- [[api-gateway/bot-auth|Боты и аутентификация]] — бот-токены, Bot API и маршруты управления
- [[api-gateway/user-status|Статусы пользователей]] — GraphQL-снимок и подписка

## Связи

- [[elph-chat-server]] — gRPC upstream для маршрутов чата
- [[elph-bot-service]] — gRPC upstream для Bot API и проверки бот-токенов
- [[elph-user-status]] — gRPC upstream для снимка статуса и источник событий подписки
