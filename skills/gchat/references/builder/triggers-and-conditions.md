# Builder: триггеры и условия

В builder есть два разных слоя триггеров:

1. Внешний триггер автоматизации запускает новый run и задаётся в метаданных `automation_create`/`automation_update`.
2. Триггер шага в `node.data.triggers[]` создаёт выход шага и соединяется ребром.

Не подменяй один слой другим. Общие правила графа — в [builder.md](../builder.md).

## Внешний триггер автоматизации

Поддерживаемые `trigger_type`: `message.received`, `client.tag.changed`, `client.tag.added`, `client.tag.removed`, `client.custom_field.changed`.

- Для `message.received` настрой `trigger_settings.mode`: `any_message` или `first_client_message`.
- Для событий тега используй `trigger_settings.tag_ids` с существующими ID из `tags_list`.
- Для `client.custom_field.changed` используй `trigger_settings.target_id` с ID поля из `custom_fields_list`.

Пример старта на каждое входящее сообщение:

```json
{
  "trigger_type": "message.received",
  "trigger_settings": { "mode": "any_message" }
}
```

## Шаговые триггеры

У каждого объекта `data.triggers[]` должны быть `id`, `type` и строковый `label`. Типы: `any_message`, `timeout`, `button_click`, `goto`, `condition`, `condition_else`.

- `any_message` — выход ожидания входящего сообщения внутри схемы; без `config`.
- `goto` — внутренний переходный выход; обычно без `config`.
- `button_click` — ожидание клика по кнопке; может содержать `config.buttonText`. Это не то же самое, что кнопка `send_message` с `action.type: "transition"`, у которой есть собственный block port.
- `timeout` — используй положительный `config.delayValue` и `config.delayUnit` (`seconds`, `minutes`, `hours`, `days`). Поле `config.minutes` — устаревший формат, только для чтения legacy-схем; не создавай его в новых схемах.
- `condition` — условный выход с `config.groups`.
- `condition_else` — выход, если ни одна условная ветка не сработала.

Для timeout можно задать `delayVariablePath` и `delayTimeWindow`. Путь должен вести к существующей числовой переменной `funnel.*` или `env.*`; если используется динамическая задержка, укажи также `delayUnit`. `delayTimeWindow` использует `HH:mm` и IANA timezone; если `enabled: true`, начало и конец не должны совпадать. Backend не проверяет полноту и положительность статической `delayValue` при сохранении схемы: проверь их сам, иначе runtime может пропустить некорректный timeout.

Используй `goto`, когда шагу нужен обычный безусловный выход к следующему шагу и у него нет подходящего выхода условия, ожидания или кнопки `transition`. У каждого ребра, исходящего из `step`, должен быть `sourceHandle`, соответствующий реальному выходу. Для `goto` это `step-trigger:<trigger.id>`; `sourceHandle: null` разрешён только для ребра из стартового узла `start`.

## Форма условия

`config.groups` — список объектов `ConditionGroup`. Каждая группа обязательно имеет `id` и непустой массив `rules`. **Не передавай группу как массив правил.** Группы объединяются через ИЛИ, правила внутри группы — через И. Пустая группа не срабатывает.

```json
{
  "id": "trigger-hi",
  "type": "condition",
  "label": "Текст равен HI",
  "config": {
    "groups": [
      {
        "id": "group-hi",
        "rules": [
          {
            "id": "rule-hi",
            "source": "variable",
            "variablePath": "sys.message.text",
            "operator": "eq",
            "value": "HI"
          }
        ]
      }
    ]
  }
}
```

Валидное правило имеет `id`, `source`, `operator` и поля, нужные для этого источника. Поля могут быть необязательными в промежуточном UI-состоянии, но не оставляй их пустыми в рабочей схеме. Поддерживаемые источники: `client_tags`, `client_custom_field`, `system_now_time`, `variable`. Для `variable` укажи namespaced `variablePath`, например `sys.message.text`, `funnel.score` или `env.REGION_CODE`. Путь не оборачивай в `{}`.

Поддерживаемые операторы: `filled`, `contains`, `contains_key`, `eq`, `gt`, `gte`, `lt`, `lte`, `before`, `after`, `starts_with`, `ends_with`, `regex`, `pattern`. `value` хранится строкой и означает сравниваемый литерал; не подставляй в него шаблон Expression Language. Например, `source: "variable"`, `variablePath: "sys.message.text"`, `operator: "eq"`, `value: "HI"` — точное сравнение текста.

Совместимость операторов зависит от источника и типа:

| Источник и тип | Операторы и значение |
|---|---|
| `client_tags` | `filled`; `contains` сравнивает с UUID тега, `contains_key` — с ключом тега |
| `client_custom_field` типа text | строковые `filled`, `eq`, `contains`, `starts_with`, `ends_with`, `regex`, `pattern` |
| `client_custom_field` типа number | `filled`, `eq`, `gt`, `gte`, `lt`, `lte`; числовой литерал передаётся строкой |
| `client_custom_field` типа select | `filled`; `eq` и `contains` проверяют выбранную строку |
| `system_now_time` | `eq`, `before`, `after`; `value` в формате `HH:mm` |
| `variable` типа string/number/boolean | `filled` и операторы, подходящие к типу; boolean `eq` принимает `true/false`, `1/0`, `yes/no`, `on/off` |

Для `client_custom_field` используй ID поля в `customFieldId`; для тега найди UUID/ключ через `tags_list`. Источник `variable` поддерживает скалярные значения; массив нельзя напрямую сравнить этим источником. Если нужна логика по массиву, сначала получи скалярное значение выражением в блоке и сравни его. Не выводи совместимость оператора только из его названия: сверяй источник и тип.

## Соединение веток

Для каждого condition-trigger создай отдельное ребро со `sourceHandle: "step-trigger:<trigger.id>"`. Для иначе-ветки создай `condition_else` и ребро от её собственного handle. Убедись, что обе ветки ведут к нужным шагам; само наличие условия или `condition_else` не отправляет ответ клиенту.

Backend проверяет общую структуру схемы не одинаково глубоко для всех триггеров. Не считай успешное сохранение доказательством того, что `groups` и `rules` имеют рабочую форму; после сохранения перечитай схему и сравни её с этим контрактом.
