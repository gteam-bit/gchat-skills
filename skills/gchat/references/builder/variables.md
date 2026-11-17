# Builder: переменные

Переменные позволяют читать контекст клиента и сообщения, хранить состояние текущего запуска и задавать переиспользуемую конфигурацию. Перед объявлением открой [общий контракт builder](../builder.md); для выражений см. [Expression Language](expression-language.md).

## Пространства имён

| Путь | Назначение | Изменение в builder |
|---|---|---|
| `sys.*` | Контекст клиента, чата, оператора, входящего сообщения и запуска | Только чтение |
| `funnel.*` | Состояние текущего запуска автоматизации | Чтение и запись блоками |
| `env.*` | Конфигурация автоматизации, повторно используемая в блоках | Только чтение |

В новых схемах всегда сохраняй полные пути с namespace. Bare-пути вроде `message.text`, `client.name` и legacy `vars.*` не создавай; старые схемы могут сохранять их для совместимости.

## Объявление в схеме

`funnel`-переменная хранит состояние конкретного запуска. `defaultValue` задаёт необязательное начальное значение:

```json
{
  "variables": {
    "funnel": [
      { "key": "lead_score", "type": "number", "description": "Оценка", "defaultValue": 0 },
      { "key": "incoming_text", "type": "string", "description": "Текст сообщения", "defaultValue": null },
      { "key": "answers", "type": "array", "description": "Ответы клиента", "defaultValue": [] }
    ],
    "env": [
      { "key": "REGION_CODE", "type": "string", "description": "Регион", "value": "eu" },
      { "key": "SUPPORTED_CHANNELS", "type": "array", "description": "Каналы", "value": ["telegram", "whatsapp"] }
    ]
  }
}
```

Поддерживаемые типы `funnel`: `string`, `number`, `boolean`, `array`. Ключ должен соответствовать `^[a-z][a-z0-9_]{0,63}$`.

Поддерживаемые типы `env`: `string`, `number`, `array`; `boolean` не поддерживается. Для `env` задаётся `value`, не `defaultValue`. Ключ должен соответствовать `^[A-Z][A-Z0-9_]{0,63}$`. Массивы окружения поддерживаются и в схеме, и в expression runtime.

Блоки, сохраняющие результат, должны ссылаться на уже объявленную переменную `funnel.*` подходящего типа. Не записывай значения в `sys.*` или `env.*`.

## Системные переменные

Пути подтверждай по доступным system-variable capabilities и по текущему контексту исполнения. Распространённые builder-пути:

- клиент: `sys.client.name`, `sys.client.id`, `sys.client.external_id`, `sys.client.phone`, `sys.client.comment`;
- пользовательское поле клиента: `sys.client.customfield.<key>`;
- чат: `sys.chat.id`, `sys.chat.status`, `sys.chat.channel`;
- оператор: `sys.operator.id`, `sys.operator.name`;
- входящее сообщение: `sys.message.id`, `sys.message.text`, `sys.message.type`, `sys.message.direction`;
- вложения: `sys.message.has_media`, `sys.message.has_text`, `sys.message.media.count`, `sys.message.media.urls`, `sys.message.media.items`;
- запуск и время: `sys.execution.id`, `sys.execution.trigger`, `sys.date`, `sys.time`, `sys.datetime`.

`sys.message.media.urls` и `sys.message.media.items` — массивы; в обычный текстовый шаблон напрямую их не вставляй. Выбери элемент или примени функцию, например `join`.

## Пути в разных полях

- В поле expression-шаблона сохраняй фигурные скобки: `{sys.message.text}`.
- В `variablePath`, `inputVariablePath`, `outputVariablePath`, `targetVariablePath` и других полях пути переменной сохраняй путь без скобок: `funnel.api_response`.
- Для шага `condition` `variablePath` — путь без скобок, а `value` — литерал для сравнения. Условие не использует шаблон `{...}`; см. [триггеры и условия](triggers-and-conditions.md).

Не объявляй системные переменные в `variables`. Для блока `set_variable` объяви цель в `variables.funnel`, затем используй её полный путь `funnel.<key>`.
