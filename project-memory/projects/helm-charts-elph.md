# helm-charts-elph

## Назначение

Содержит Helm-конфигурацию сервисов и общего Redis.

## Устройство

- Chart Redis при включённой персистентности подключает постоянный том и включает AOF с `appendfsync everysec`; без персистентности сохранение отключено
- В ветке `refs_550675` Chart bot-service передаёт параметры PostgreSQL отдельными переменными, а пароль получает через `bot-service-secret`; Chart PostgreSQL создаёт базу и пользователя `bot_service`

## Особенности и риски

- Конфигурация Chart не задаёт резервное копирование Redis. Наличие внешнего резервного копирования по репозиторию не установлено

## Связи

- [[elph-bot-service]] — использует общий Redis для подписок управляемых ботов и отдельную базу PostgreSQL для токенов

## Источники

- `redis/templates/deployment.yaml`, `redis/templates/configmap.yaml`, `redis/values.yaml`; `bot-service/values.yaml`, проверено по `origin/develop/3.9.0` 2026-11-09
- `bot-service/values.yaml`; `postgres/values.yaml`; `secrets.sh`, проверено по `refs_550675` 2026-11-12
