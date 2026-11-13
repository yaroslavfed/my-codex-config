# Рабочие статусы УРВ

## Проверено в ветке `refs_556137`

- `UrvExternalService.getUrvDepartmentEmployees` получает `EmployeesList` (или мок при `URV_MOCK_DATA=true`), а его дерево сотрудников содержит `HRMid` и `mail`
- `UsersService.getByHrmId` ищет пользователя в БД Jarvis по `hrm_id`
- `WorkStatusesPollingService` раз в `URV_POOL_INTERVAL` получает дневные данные УРВ, формирует полный снимок и публикует его в Redis-канал `urv.work-statuses.snapshot.v1` только при успешном завершении всех шагов
- При пустом `REDIS_URL` опрос рабочих статусов не запускается, поэтому офисный поток остаётся выключенным в обычной поставке
- При `URV_MOCK_DATA=true` справочник `EmployeesList` содержит отдельную группу из 28 сотрудников с непустыми `HRMid`, которая покрывает карту рабочих статусов, приоритеты и неизвестные причины. Интеграционный тест поднимает Redis в контейнере и проверяет полный путь до публикации

## Сценарий рабочих статусов

- Модуль Jarvis последовательно запрашивает `EmployeeLocation` и `AbsenteeismOfEmployees` за одну дату, выбирает один статус и публикует полный снимок в Redis Pub/Sub для [[elph-user-status/urv-work-statuses|elph-user-status]]
- Сообщение содержит `schemaVersion`, `source`, `date`, `polledAt` и подготовленный массив пользователей. Jarvis не передаёт `domain` и не обращается к profile-service для сопоставления с Elph

## Источники

- `src/urv/urv-external.service.ts`, `UrvExternalService.getUrvDepartmentEmployees`, `processDepartmentEmployees`; `src/db/services/users.service.ts`, `UsersService.getByHrmId`; `src/work-statuses/work-statuses-polling.service.ts`, `WorkStatusesPollingService`; `src/work-statuses/work-statuses-redis.publisher.ts`, `WorkStatusesRedisPublisher`; `src/work-statuses/work-statuses-redis.integration.spec.ts`, проверено в ветке `refs_556137` 2026-11-13
- Целевой поток: `/home/iaroslavfed/Downloads/Принцип работы статусов УРВ-v3-20261112_142904.pdf`, редакция 2026-11-12
