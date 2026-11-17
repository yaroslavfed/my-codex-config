---
type: topic
---

# Аватары

Маршруты (`src/services/file-service/avatars.ts`, `registerAvatars`):

- `GET /elph_chat/avatar/user/:userId` и `GET /elph_chat/avatar/:username` - аватар пользователя
- `GET /elph_chat/avatar/room/:roomId` - описание от chat-server `GetAvatar`, при `userId` в ответе
  подставляется аватар этого пользователя
- `/elph_api/loadLocal/*`, `/api/avatar/*`, `/elph_api/loadRemote/` - прямой прокси по описанию

Все ответы собираются в `AvatarDescription` и проксируются в file-service
`GET /internal/avatars/image` с описанием в заголовке `X-Avatar-Description` (JSON в base64url),
личность пробрасывается через `buildIdentityHeaders`. При ошибке >= 400 по основному описанию
один раз повторяется fallback с `unavailable: true`

Кросс-доменная проверка в `userImage`: аватар пользователя чужого домена заменяется буквами.
Ветка `refs_556479` (feature #556479) добавила исключение для `identityKind(identity) === 'system'`
(`src/platform/auth/identity-kind.ts`): системные учётки (`SYSTEM_USER_IDENTITIES` в
platform-contracts, домен `system`, например security-user и supportuser) видят аватары
пользователей любого домена, исключение общее для всех системных учёток, а не только security-user

Карточки пользователей кэшируются в Redis ключами `gw-avatar:*` с TTL 10 минут
(`src/services/profile/user-cache.ts`), инвалидация по событию profile

Источники: `src/services/file-service/avatars.ts`, `src/services/profile/user-cache.ts`,
`src/platform/auth/identity-kind.ts`, проверено 2026-11-16 на ветке refs_556479 и живом стенде
