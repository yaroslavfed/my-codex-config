---
type: module
---

# Рабочие статусы УРВ

## Текущее состояние

- В ветке `refs_556140` рабочие статусы УРВ реализованы. Jarvis публикует полный снимок в Redis-канал `urv.work-statuses.snapshot.v1`, а сервис статусов проверяет его схему, сопоставляет полный email с пользователями `profile-service` и хранит один статус на `userId`
- Функция включается только `URV_WORK_STATUS_ENABLED=true`. При отсутствии или другом значении сервис не подписывается на снимки и не выдаёт сохранённый рабочий статус
- Справочник `profile-service` включает пользователей всех доменов, но пропускает пустые email и domain. Один email может соответствовать нескольким `User.id`, и снимок применяется ко всем таким пользователям
- В `WorkStatuses` хранятся `code`, исходные `startsAt` и `endsAt`, а также внутренний `expiresAt`. Полный снимок упорядочивается по глобальному `polledAt`; повтор продлевает TTL без события, устаревший снимок не применяется
- `GetPresence` и `UserStatusChangedEvent` содержат `workStatus`. Для пользователя без `ElphUsers` возвращаются значения presence по умолчанию и домен профиля. Если строка `ElphUsers` есть, её domain используется для ответа и события

## Контракт и поток

- Рабочий статус независим от сетевого/DND и текстового. В GraphQL `aggregateElphUserStatuses` и `elphUserStatusChanged` есть nullable поле `workStatus` с `code`, `startsAt`, `endsAt`; отсутствие, удаление или истечение TTL даёт `null`
- TTL задаёт `WORK_STATUS_TTL_SECONDS`, по умолчанию 1800 секунд. Очистка запускается раз в две минуты. Продление неизменного статуса не создаёт событие
- Gateway сохраняет свою доменную изоляцию при query и WS. Рабочий статус назначается без фильтра домена, но виден только в домене пользователя
- Карта клиентских кодов: `office`, `outside_office`, `remote_work`, `gym`, `smoking`, `annual_leave`, `study_leave`, `parental_leave`, `unpaid_leave`, `day_off`, `sick_leave`, `business_trip`. `other_absence` и `no_data` исключены

## Ограничения

- Redis Pub/Sub не хранит пропущенные снимки. Следующий полный снимок восстанавливает актуальное состояние, а при длительном отсутствии Jarvis или УРВ записи удаляются по TTL
- Переход миграции к одному статусу на `userId` прекращается до изменения таблиц, если в старых данных один пользователь есть в нескольких доменах. Для такого состояния нет правила выбора значения

## Связи

- [[platform-contracts]] — gRPC `UserPresence` и Redis `UserStatusChangedEvent` должны получить рабочий статус

## Источники

- `src/work-statuses/`, `src/grpc/presence-registry.service.ts`, `src/users/users.watcher.ts`, ветка `refs_556140`, проверено 2026-11-19
- План: `/home/iaroslavfed/Downloads/Принцип работы статусов УРВ-v3-20261112_142904.pdf`, `/home/iaroslavfed/Downloads/Клиентский контракт рабочих статусов УРВ-v11-20261112_142913.pdf`, `/home/iaroslavfed/Downloads/Статусы УРВ-v4-20261112_142843.pdf`, редакции 2026-11-12
- Локальная проработка для сверки: `docs/urv-work-statuses-implementation.md`, `docs/urv-work-statuses-client-contract.md`, `docs/urv-work-statuses-business-requirements.md`, untracked в `develop/3.7.0`, проверено 2026-11-12
