# Рабочие статусы УРВ

## Проверено в ветке `feature-new-sdk-transport`

- `UrvExternalService.getUrvDepartmentEmployees` получает `EmployeesList` (или мок при `URV_MOCK_DATA=true`), а его дерево сотрудников содержит `HRMid` и `mail`
- `UsersService.getByHrmId` ищет пользователя в БД Jarvis по `hrm_id`

## Целевой сценарий

- По проекту рабочих статусов от 2026-11-12 отдельный модуль Jarvis должен последовательно запрашивать `EmployeeLocation` и `AbsenteeismOfEmployees` за одну дату, выбирать один статус и публиковать полный снимок в Redis Pub/Sub для [[elph-user-status/urv-work-statuses|elph-user-status]]
- Этот поток ещё не реализован; Jarvis не обращается к profile-service для сопоставления с Elph

## Источники

- `src/urv/urv-external.service.ts`, `UrvExternalService.getUrvDepartmentEmployees`, `processDepartmentEmployees`; `src/db/services/users.service.ts`, `UsersService.getByHrmId`, проверено в локальной ветке `feature-new-sdk-transport` 2026-11-12
- Целевой поток: `/home/iaroslavfed/Downloads/Принцип работы статусов УРВ-v3-20261112_142904.pdf`, редакция 2026-11-12
