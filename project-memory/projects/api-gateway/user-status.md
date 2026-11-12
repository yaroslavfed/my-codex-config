# Статусы пользователей

## Проверено в ветке `refs_550675`

- `aggregateElphUserStatuses(userIds)` и `elphUserStatusChanged(userIds)` получают снимок и события из [[elph-user-status]]

## Целевой контракт

- По проекту рабочих статусов от 2026-11-12 эти же операции должны вернуть третье nullable поле `workStatus`; в проверенных GraphQL SDL и мапперах поля пока нет
- Подробности сценария: [[elph-user-status/urv-work-statuses|Рабочие статусы УРВ]]

## Источники

- `src/services/user-status/graphql/schema.ts`, `resolvers.ts`, `mappers.ts`, `aggregateElphUserStatuses`, `elphUserStatusChanged`, локальная ветка `refs_550675`, проверено 2026-11-12; символы найдены WebStorm index
- Целевой контракт: `/home/iaroslavfed/Downloads/Клиентский контракт рабочих статусов УРВ-v11-20261112_142913.pdf`, редакция 2026-11-12
