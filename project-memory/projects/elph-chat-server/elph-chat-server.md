# elph-chat-server

Сервис хранения и обработки чата

## Архитектура

[[elph-chat-server.canvas|Схема elph-chat-server]]

## Темы

- [[elph-chat-server/messages|Сообщения]] — отправка, дельта обновлений и нормализация
- [[elph-chat-server/inline-commands|Inline-команды]] — исполнение команд и порядок событий
- [[elph-chat-server/bot-tokens|Бот-токены]] — выдача, проверка и выгрузка токенов

## Связи

- [[api-gateway]] — публичные HTTP-маршруты преобразуются в gRPC-вызовы этого сервиса
- [[elph-bot-service]] — получает события сообщений и выгружает ранее выданные токены
