# elph-bot-jarvis

## Назначение

Бот Jarvis принимает сообщение пользователя и выполняет запрос к ИИ-сервису через потоковый gRPC-контракт.

## Устройство

- `AiService` отправляет «Думаю...», хранит состояние активной попытки и данные для повтора в памяти процесса
- События `progress` и `tool` не меняют сообщение; `final_answer` заменяет его ответом, `error` показывает ошибку и предлагает отдельную inline-кнопку «Повторить»
- При неудаче редактирования сообщения предусмотрен запасной способ доставки ответа новым сообщением; состояние повтора не является долговечным хранилищем

## Контракты и сценарии

- `TaskService.ExecuteTask` принимает один `TaskRequest` и возвращает поток `TaskResponse`
- Запрос содержит `request_id`, `rid`, `username`, `message`; ответ повторяет `request_id`, `rid` и содержит один вариант события: `progress`, `tool`, `final_answer` или `error`
- `question_id` в проверенном proto-контракте нет
- Действующий `UrvExternalService.getUrvDepartmentEmployees` получает `EmployeesList` (или мок при `URV_MOCK_DATA=true`), а его дерево сотрудников содержит `HRMid` и `mail`. `UsersService.getByHrmId` ищет пользователя в БД Jarvis по `hrm_id`.
- По проекту рабочих статусов от 2026-11-12 отдельный модуль Jarvis должен последовательно запрашивать `EmployeeLocation` и `AbsenteeismOfEmployees` за одну дату, выбирать один статус и публиковать полный снимок в Redis Pub/Sub для [[elph-user-status]]. Этот поток ещё не реализован; Jarvis не обращается к profile-service для сопоставления с Elph.

## Связи

- [[elph-user-status]] — планируемый получатель готовых рабочих статусов УРВ из офисного Jarvis.

## Источники

- `src/ai/proto/ai_integration.proto`, `TaskService.ExecuteTask`, проверено в локальной ветке `refs_551774` 2026-11-09
- `src/ai/ai.service.ts`, `AiService.runAttempt`, `handleResponse`, `showError`, `replaceThinkingMessage`, проверено в локальной ветке `refs_551774` 2026-11-09
- `src/urv/urv-external.service.ts`, `UrvExternalService.getUrvDepartmentEmployees`, `processDepartmentEmployees`; `src/db/services/users.service.ts`, `UsersService.getByHrmId`, проверено в локальной ветке `feature-new-sdk-transport` 2026-11-12
- Целевой поток: `/home/iaroslavfed/Downloads/Принцип работы статусов УРВ-v3-20261112_142904.pdf`, редакция 2026-11-12
