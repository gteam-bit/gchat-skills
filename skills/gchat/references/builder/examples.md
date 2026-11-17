# Builder: короткие примеры

В этом файле собраны только фрагменты блоков и небольшие связки. Общий формат схемы описан в [контракте builder](../builder.md), правила размещения узлов — в [размещении узлов](layout.md), условия — в [триггерах и условиях](triggers-and-conditions.md), выражения — в [Expression Language](expression-language.md).

ID примеров локальны. UUID внешних ресурсов получай через GChat MCP или от пользователя. Не выдавай фрагмент за готовую схему автоматизации.

## Условие: `HI` или иначе

Фрагмент триггера сравнивает текст точно и с учётом регистра. `value` — литерал, не expression. Группа имеет вид `{id, rules}`:

```json
{
  "id": "trigger-hi",
  "type": "condition",
  "label": "Текст равен HI",
  "config": {
    "groups": [{
      "id": "group-hi",
      "rules": [{
        "id": "rule-hi",
        "source": "variable",
        "variablePath": "sys.message.text",
        "operator": "eq",
        "value": "HI"
      }]
    }]
  }
}
```

Свяжи выход `trigger-hi` с шагом, где `send_message.text` равен `1`; выход `condition_else` ведёт на отдельный шаг с ответом `0`. Расположи ветви на разных строках, чтобы направление потока читалось без пересечения линий.

```json
[
  { "id": "reply-one", "type": "send_message", "text": "1", "pinMessage": false, "buttons": [] },
  { "id": "reply-zero", "type": "send_message", "text": "0", "pinMessage": false, "buttons": [] }
]
```

## Кнопочное меню: block port → шаг

В menu-схеме ребро строится от конкретной кнопки с `action.type: "transition"`. Располагай цель справа от сообщения и оставляй место под соседние кнопочные ветви:

```json
{
  "nodes": [
    {
      "id": "step-menu",
      "type": "step",
      "position": { "x": 360, "y": 120 },
      "data": {
        "title": "Разделы",
        "blocks": [{
          "id": "block-menu",
          "type": "send_message",
          "text": "Выберите раздел",
          "pinMessage": false,
          "buttons": [{
            "id": "button-catalog",
            "key": "catalog",
            "label": "Каталог",
            "color": "green",
            "action": { "type": "transition" }
          }]
        }],
        "triggers": []
      }
    },
    {
      "id": "step-catalog",
      "type": "step",
      "position": { "x": 900, "y": 120 },
      "data": { "title": "Каталог", "blocks": [], "triggers": [] }
    }
  ],
  "edges": [{
    "id": "edge-menu-catalog",
    "source": "step-menu",
    "target": "step-catalog",
    "sourceHandle": "block-port:send_message:block-menu:catalog",
    "targetHandle": null,
    "type": "step"
  }]
}
```

Это фрагмент двух связанных шагов, не полная схема: start-узел, соседние кнопки и другие действия здесь опущены. `sourceHandle` должен совпадать с ID блока и `key` кнопки.

## Ожидание ответа или таймаут

В референсе один ожидающий шаг имеет два самостоятельных выхода. Разнеси их по разным строкам и привяжи каждое ребро к своему trigger ID:

```json
{
  "triggers": [
    { "id": "trigger-reply", "type": "any_message", "label": "Ответ клиента" },
    {
      "id": "trigger-timeout",
      "type": "timeout",
      "label": "Ответ не пришёл",
      "config": { "delayValue": 1, "delayUnit": "hours" }
    }
  ],
  "edges": [
    {
      "id": "edge-reply",
      "source": "step-wait",
      "target": "step-process-reply",
      "sourceHandle": "step-trigger:trigger-reply",
      "targetHandle": null,
      "type": "step"
    },
    {
      "id": "edge-timeout",
      "source": "step-wait",
      "target": "step-remind",
      "sourceHandle": "step-trigger:trigger-timeout",
      "targetHandle": null,
      "type": "step"
    }
  ]
}
```

Целевые шаги `step-process-reply` и `step-remind` должны быть отдельными узлами схемы; общий следующий этап соединяй с ними только в настоящей точке схождения.

## Повторяющиеся проверки и дорожки

В проверочном сценарии из референсов повторяется короткая связка: запрос/проверка → condition → отдельные продолжения для совпадения и `else`. Назови шаги по проверяемому признаку, например `Проверить ID`, `ID найден`, `ID отсутствует`; не копируй неоднозначные `Step 2`/`Step 5`.

В большом retention-сценарии повторяется другая форма: роутер стадии → несколько шагов обработки → сбор ответа → оценка результата → уточнение или завершение. Располагай одинаковые стадии вертикальными дорожками, держи соответствующие этапы в одинаковых колонках и используй короткие `note` для крупных групп, когда это помогает ориентироваться. Не копируй полный граф или реальные ID из экспорта.

## Сохранить входящий текст

Переменная `funnel.incoming_text` должна быть объявлена в схеме. У пути целевой переменной нет фигурных скобок, а шаблонное значение использует их:

```json
{
  "id": "block-store-message",
  "type": "set_variable",
  "targetVariablePath": "funnel.incoming_text",
  "value": "{sys.message.text}"
}
```

## Добавить заметку на схему

`note` помогает подписать область или пояснить ветку. Это визуальная нода: она не участвует в исполнении, не соединяется рёбрами и отображается за рабочими узлами. Расположи её рядом с поясняемыми шагами так, чтобы текст не закрывал порты и подписи рёбер.

```json
{
  "id": "note-hi-branch",
  "type": "note",
  "position": { "x": 1120, "y": 0 },
  "width": 560,
  "height": 520,
  "data": {
    "color": "info",
    "title": "Секция: ответы по ветвям"
  }
}
```

Допустимые цвета: `info`, `success`, `warning`, `destructive`. Размер выбирай по реальной группе: note может быть небольшой пометкой или крупной фоновой секцией. Ставь её рядом с относящимися к ней шагами и не считай заменой узла `step`.

## HTTP-запрос с ответом в переменную

Объяви строковую переменную `funnel.api_response`. Пример показывает только содержимое блока; перед использованием укажи реальный URL и проверь обязательные поля и нужный тип тела.

```json
{
  "id": "block-send-api-request",
  "type": "http_request",
  "method": "post",
  "url": "https://api.example.invalid/leads",
  "headers": [],
  "queryParams": [],
  "bodyType": "json",
  "jsonBody": "{\"client_id\":\"{sys.client.id}\"}",
  "rawBody": "",
  "formData": [],
  "urlEncodedBody": [],
  "outputVariablePath": "funnel.api_response"
}
```

Ответ сохраняется как текст. Для обращения к полю JSON используй, например, `{jsonDecode(funnel.api_response).status}`.
