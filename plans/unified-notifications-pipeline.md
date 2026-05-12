# Унифицированный pipeline уведомлений (с каналом chat_room)

## Проблема

Сейчас realtime‑события публикуются через **два параллельных механизма**:

1. `NotificationEventPublisher` (backend) / `NotificationsPublisherService` (cron‑worker)
   → персистит `notification_events` / `notification_recipients` /
   `notification_deliveries`, рассылает по каналам
   `in_app | ws | sms | push | email`, имеет Redis‑bridge
   (`NotificationsRealtimeBridge`) для cron‑worker → backend WS.
2. `DisputesChatGateway.emit(...)` — прямой вызов
   `server.to(room).emit(...)` из `DisputesService` и `AdminDisputesService`.
   Без персистенции, без аудита, без recovery, без доступа из cron‑worker.

Следствия:

- `DisputesService.sendMessage` пишет строку чата в БД, потом
  **отдельно** вызывает gateway, потом **отдельно** вызывает
  notifications publisher — три вызова, которые могут рассинхронизироваться
  при частичном сбое.
- Cron‑worker не может постить системные сообщения в тред диспута
  (в worker‑процессе нет socket.io‑сервера).
- События чата неаудитируемы и не ретраятся, а inbox‑события — да,
  без принципиальной причины для такого различия.

## Цель

Одна публичная точка входа — `NotificationPublisherService.publish(input)` —
которая внутри маршрутизирует событие по **подключаемым каналам**.
`DisputesChatGateway` становится каналом `chat_room` наравне с
`in_app`, `ws`, `sms`, `push`, `email`.

Из целей **исключено**:

- Слияние двух WS‑gateway в один namespace (см. отдельное обоснование вне плана).
- Изменение WS‑контракта на фронте (`/disputes-chat` и `/notifications`
  сохраняют свои имена событий и payload‑ы).
- Изменение хранения чата (`dispute_chat_messages` остаётся источником
  истины для истории чата; REST‑эндпоинты истории не меняются).

## Дизайн

### Таксономия каналов

Расширяем `NotificationsChannel`:

```ts
export type NotificationsChannel =
  | 'in_app'      // персистентный + ws‑inbox
  | 'ws'          // только ws (без persist) — существующий
  | 'chat_room'   // НОВЫЙ — эфемерный, адресация по комнате, internal‑only
  | 'sms'
  | 'push'
  | 'email';
```

У каждого канала декларируются возможности:

```ts
interface ChannelDescriptor {
  key: NotificationsChannel;
  persistent: boolean;       // пишет ли в notification_events / recipients / deliveries
  recoverable: boolean;      // подхватывается ли recovery scan'ом NotificationDeliveryWorker
  addressing: 'user' | 'room';
  internalOnly?: boolean;    // запрещает publish() из request‑scoped контроллеров
}
```

`chat_room`:
`{ persistent: false, recoverable: false, addressing: 'room', internalOnly: true }`.

### Адресация получателей

Заменяем неявный `{ role, recipientId }` на размеченный union:

```ts
type RecipientTarget =
  | { kind: 'user'; role: NotificationsRecipientRole; recipientId: string }
  | { kind: 'room'; namespace: 'disputes-chat'; roomKey: string };
```

Loose‑ и known‑варианты publish input принимают такой target.
Канал берёт только тех получателей, у которых `kind` совпадает с его
`addressing`.

### Разделение pipeline

Текущий `NotificationPublishPipelineService.publish` делает
`persist + dispatch` в одной транзакции. Делим на две фазы:

1. **Persist phase** — только для получателей, чьи каналы включают
   хотя бы один `persistent: true`. Создаёт `eventId`, строки recipients,
   очерёдные строки deliveries.
2. **Dispatch phase** — для каждого канала из резолвнутого набора:
   - `in_app` / `ws` → существующий путь
     (`NotificationsGateway.emitNotificationCreated`).
   - `chat_room` → новый `ChatRoomChannelDispatcher` (см. ниже).
     Без записи в БД, без строки delivery.
   - `sms` / `push` / `email` → без изменений.

Pipeline остаётся одним методом; каналы диспатчатся через
`Map<NotificationsChannel, ChannelDispatcher>`.

### ChatRoomChannel

Backend‑реализация:

```ts
@Injectable()
export class ChatRoomChannelDispatcher implements ChannelDispatcher {
  constructor(private readonly chatGateway: DisputesChatGateway) {}

  async dispatch(event: ResolvedEvent, target: RoomTarget): Promise<void> {
    if (target.namespace !== 'disputes-chat') return;
    this.chatGateway.emitToRoom(target.roomKey, event.eventType, event.payload);
  }
}
```

Cron‑worker использует Redis pub/sub (зеркало
`NotificationsRealtimeBridge`):

- worker‑овский `ChatRoomChannelDispatcher` публикует в новый канал
  `CHAT_ROOM_WS_EMIT_CHANNEL`;
- backend подписывается через `DisputesChatRealtimeBridge` (новый,
  симметричный `NotificationsRealtimeBridge`) и форвардит в
  `DisputesChatGateway.emitFromBridge(roomKey, event, payload)`
  (новый метод, симметричный `NotificationsGateway.emitFromBridge`).

### Изменения в registry событий

Каждое событие в `INotificationsEventRegistry` декларирует свои
**каналы по умолчанию** и допустимые room‑targets:

```ts
'dispute.message.created': {
  domain: 'disputes',
  sourceEntityType: 'dispute',
  defaultChannels: ['chat_room', 'in_app', 'ws', 'push'],
  payload: IDisputeMessageCreatedPayload,
  recipients: { ... },
  rooms: ['disputes-chat'],   // допускаются chat_room targets для этого события
};
```

`publish()` резолвит каналы как объединение caller‑provided ∪ defaults
из registry. Защита: канал не может использоваться для события,
которое его не декларирует (compile‑time + runtime check).

### Удаление прямых вызовов gateway

После того как pipeline на месте:

- `DisputesService.sendMessage` перестаёт звать `gateway.emit(...)` и
  перестаёт отдельно звать `notificationPublisher.publish(...)` для
  того же бизнес‑события. Один вызов:

  ```ts
  await notificationPublisher.publish({
    eventType: 'dispute.message.created',
    payload: { ..., dispute_pk },
    recipients: [
      { kind: 'room', namespace: 'disputes-chat', roomKey: dispute_pk },
      { kind: 'user', role: 'client',   recipientId, body: ... },
      { kind: 'user', role: 'merchant', recipientId, body: ... },
    ],
  });
  ```

- То же самое для `AdminDisputesService` (системные сообщения —
  только recipient типа `chat_room`).
- В cron‑worker'е `dispute-sla.job.ts` теперь может также добавлять
  `chat_room`‑recipient'а при отправке SLA‑предупреждений в тред.

### Что **не** меняется

- Таблица `dispute_chat_messages` и её запись из `DisputesService`/
  `AdminDisputesService` остаются как есть. Это источник истины для
  истории чата.
- REST‑эндпоинты истории диспутов не меняются.
- WS‑события для фронта (`dispute.chat.message.created`,
  `dispute.chat.message.system`, `notification.created`) сохраняют
  имена и payload‑ы.

## Поэтапный rollout (todos)

1. **Фундамент**
   - Ввести интерфейс `ChannelDispatcher` + registry в
     `core/notifications/services/`.
   - Разделить `NotificationPublishPipelineService` на фазы persist
     и dispatch. Сохранить поведение существующих каналов 1‑в‑1.
   - Добавить тесты, фиксирующие текущую семантику `in_app`/`ws`
     (персистенция, recovery, ws emit).

2. **Рефактор RecipientTarget**
   - Ввести размеченный union `RecipientTarget`. Существующие caller'ы
     по умолчанию получают `kind: 'user'`. Поведение не меняется.

3. **Канал chat_room — backend**
   - Добавить `'chat_room'` в `NotificationsChannel`.
   - Реализовать `ChatRoomChannelDispatcher` в backend
     `core/notifications`.
   - Добавить публичный метод `DisputesChatGateway.emitToRoom(roomKey,
     type, payload)` — internal‑only, типизированный по registry.
   - Зарегистрировать диспатчер в registry каналов.

4. **Миграция `DisputesService.sendMessage`**
   - Заменить прямой `gateway.emit` + отдельный publish уведомления
     на один вызов `publish` с recipients типа `room` и `user`
     одновременно.
   - Убедиться, что `disputes-chat.realtime.spec.ts` и
     `typed-emit-api.spec.ts` проходят (или обновить ассерты на путь
     dispatch'а, не на контракт).

5. **Миграция `AdminDisputesService` (системные сообщения)**
   - То же самое: один `publish` с recipient'ом типа `chat_room`.

6. **chat_room транспорт в cron‑worker**
   - Добавить константу `CHAT_ROOM_WS_EMIT_CHANNEL` в
     `core/notifications/notifications.constants.ts`.
   - Реализовать в worker'е `ChatRoomChannelDispatcher`,
     публикующий в Redis.
   - Реализовать на backend `DisputesChatRealtimeBridge` (зеркало
     `NotificationsRealtimeBridge`), который подписывается и
     форвардит в `DisputesChatGateway.emitFromBridge(...)`.
   - Обновить `dispute-sla.job.ts`: опционально добавлять
     `chat_room`‑recipient'а для системных сообщений в тред.

7. **Hardening**
   - Добавить guard `internalOnly`: отказывать в `publish()` с
     `kind: 'room'`, если вызов исходит из request‑scope контроллера
     (публиковать в комнаты вправе только сервисы).
   - Описать семантику `chat_room` в
     `core/disputes/ws/EVENTS.md` и в README `core/notifications`.
   - Удалить теперь неиспользуемые прямые `gateway.emit` экспорты,
     если caller'ов не осталось (gateway сохраняем, сужаем
     публичную поверхность).

## Критерии приёмки

- `DisputesService.sendMessage` делает ровно один вызов
  `publish(...)` на одно бизнес‑событие.
- Job в cron‑worker'е может отправить системное сообщение в тред
  диспута, и аутентифицированный клиент, подписанный на
  `/disputes-chat`, его получает.
- В таблице `notification_events` не появляется строка на каждое
  сообщение чата.
- Существующее поведение уведомлений (recovery scan, ретраи,
  multi‑channel fan‑out) для `in_app`/`ws`/`sms`/`push`/`email`
  не меняется.
- `npm run verify` проходит и в `app/backend`, и в `app/cron-worker`.

## Риски и митигации

- **Риск:** разделение pipeline регрессирует семантику персистенции.
  **Митигация:** зафиксировать текущее поведение тестами **до**
  рефактора; набор каналов по умолчанию в registry остаётся
  идентичным текущему.
- **Риск:** `chat_room` случайно становится recoverable/persistent
  из‑за caller'а, передавшего `channels: ['chat_room', 'in_app']`
  для не‑room события.
  **Митигация:** registry валидирует декларацию `event.rooms`;
  runtime отвергает несовпадающие targets.
- **Риск:** worker → backend bridge молча теряет сообщения при
  падении Redis.
  **Митигация:** тот же trade‑off, что и у текущего
  notifications‑bridge — задокументировано как best‑effort.
  Персистентный fan‑out (`in_app`) обеспечивает durable‑доставку;
  тред чата — best‑effort по своей природе.
