# Запросы к ИИ

## Проверено в ветке `refs_551774`

- `AiService` отправляет «Думаю...», хранит состояние активной попытки и данные для повтора в памяти процесса
- События `progress` и `tool` не меняют сообщение; `final_answer` заменяет его ответом, `error` показывает ошибку и предлагает отдельную inline-кнопку «Повторить»
- При неудаче редактирования сообщения предусмотрен запасной способ доставки ответа новым сообщением; состояние повтора не является долговечным хранилищем
- `TaskService.ExecuteTask` принимает один `TaskRequest` и возвращает поток `TaskResponse`
- Запрос содержит `request_id`, `rid`, `username`, `message`; ответ повторяет `request_id`, `rid` и содержит один вариант события: `progress`, `tool`, `final_answer` или `error`
- `question_id` в проверенном proto-контракте нет

## Источники

- `src/ai/proto/ai_integration.proto`, `TaskService.ExecuteTask`, проверено в локальной ветке `refs_551774` 2026-11-09
- `src/ai/ai.service.ts`, `AiService.runAttempt`, `handleResponse`, `showError`, `replaceThinkingMessage`, проверено в локальной ветке `refs_551774` 2026-11-09
