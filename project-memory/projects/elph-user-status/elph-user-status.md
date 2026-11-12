# elph-user-status

Сервис хранит состояния пользователей Elph, ведёт presence и отдаёт их API gateway по gRPC

## Темы

- [[elph-user-status/presence|Presence]] — текущие состояния и доставка изменений
- [[elph-user-status/urv-work-statuses|Рабочие статусы УРВ]] — целевой контракт и открытые вопросы

## Связи

- [[api-gateway]] — текущий gRPC-клиент и внешний GraphQL-вход
- [[elph-bot-jarvis]] — планируемый издатель полных снимков УРВ
- [[platform-contracts]] — источник gRPC и Redis-контрактов статуса
