# helm-charts-elph

## Назначение

Содержит Helm-конфигурацию сервисов и общего Redis.

## Устройство

- Chart Redis при включённой персистентности подключает постоянный том и включает AOF с `appendfsync everysec`; без персистентности сохранение отключено
- Chart bot-service передаёт подключение к Redis через существующий Secret

## Особенности и риски

- Конфигурация Chart не задаёт резервное копирование Redis. Наличие внешнего резервного копирования по репозиторию не установлено

## Связи

- [[elph-bot-service]] — использует общий Redis для подписок управляемых ботов

## Источники

- `redis/templates/deployment.yaml`, `redis/templates/configmap.yaml`, `redis/values.yaml`; `bot-service/values.yaml`, проверено по `origin/develop/3.9.0` 2026-11-09
