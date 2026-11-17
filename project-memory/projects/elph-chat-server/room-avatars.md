---
type: topic
---

# Аватары комнат

Поток: клиент GET `gateway /elph_chat/avatar/room/:roomId` → gRPC `RoomsService.GetAvatar`
(`src/grpc/rooms.grpc.controller.ts`) → `RoomsService.getAvatar` (`src/api/rooms/rooms.service.ts`)
→ `describeRoomAvatar` (`src/api/rooms/room-avatar.ts`). Сервис читает только projection комнаты
и отвечает описанием (`location`, `avatarETag`, `fallback`, опционально `userId` или `staticImage`),
байты отрисовывает file-service через gateway

Правила доступа в `describeRoomAvatar`: 403 `no-access-to-room` только для директа (`t='d'`) и
звонка (`t='tc'` + `meetingInfo.meetingType='call'`), когда вызывающего нет в `uids`. Групповые
комнаты такой проверки не имеют. Для директа из двух участников в `userId` ответа попадает
собеседник (`uids.find(id => id !== callerId)`), поэтому вызывающему вне `uids` отдаётся первый
участник из `uids`. Директ с одним uid (сохранённые сообщения) отдаёт staticImage SAVED_MESSAGES

Ветка `refs_556479` (feature #556479): исключение из 403 для `SECURITY_USER._id` из
`@elph/platform-contracts` (проверено 2026-11-16 на живом стенде). Проверка по id учётки,
а не по роли `security-agent`: персональная учётка с ролью security-agent и другим `_id`
403 по-прежнему получает. Роль security-agent возможна и у обычных пользователей справочника
(`toChatRoles` в `src/authorization/chat-roles.ts`, `User.isSecurityAgent` в
`src/database/models/abstract/Users/User.ts`, сценарии папок безопасника в
`src/api/rooms/roomsSync.service.ts`)

Удалённая комната (`room = null`) отдаёт fallback по roomId без 403, ролевой модели для неё
на момент проверки нет (P4 в требованиях #556479)

Источники: `src/api/rooms/room-avatar.ts`, `src/grpc/rooms.grpc.controller.ts`,
`src/api/rooms/rooms.service.ts`, проверено 2026-11-16 на ветке refs_556479 и живом стенде
(mongo + redis + rabbitmq + gateway + profile-service + file-service)
