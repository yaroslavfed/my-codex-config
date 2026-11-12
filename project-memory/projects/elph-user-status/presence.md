# Presence

## Текущее поведение

- Сервис использует PostgreSQL (`ElphUsers`, кастомные статусы, аккаунты), Redis для реестра соединений и публикации событий, а также PostgreSQL NOTIFY и RabbitMQ для существующих потоков
- `UserStatusService.GetPresence` возвращает снимок присутствия. `UsersWatcher` преобразует изменения `ElphUsers` в событие `UserStatusChangedEvent` для Redis-канала, который слушает [[api-gateway/user-status|api-gateway]]

## Источники

- `src/app.module.ts`, `src/users/users.entity.ts`, `src/grpc/user-status.controller.ts`, `src/grpc/user-status.mapper.ts`, `src/users/users.watcher.ts`, `src/subscriptions/user-status-redis.publisher.ts`, локальная ветка `develop/3.7.0`, проверено 2026-11-12
