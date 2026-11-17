# Builder: Expression Language

Открой этот файл для полей, которые исполняют выражения или шаблоны. Для namespaces и объявления переменных см. [переменные](variables.md). Для `condition`-триггера см. [триггеры и условия](triggers-and-conditions.md): его `value` — литерал, а не expression.

## Синтаксис шаблонов

Выражение пишется в `{...}`. Одна expression занимает всё поле и сохраняет исходный тип результата; expression внутри обычного текста подставляется как текст.

```text
Привет, {sys.client.name}!
{funnel.order_total}
{lowercase(sys.message.text)}
```

В смешанном текстовом шаблоне допустимы простые значения: строка, число, boolean или `null`. `null` превращается в пустой текст; `true`/`false` — в соответствующие слова. Массив или объект нельзя вставлять напрямую в обычный текст: сначала выбери скалярное поле или преобразуй массив, например `join(sys.message.media.urls, ", ")`.

Для выражения в `set_variable.value` используй template-форму:

```json
{
  "id": "block-copy-text",
  "type": "set_variable",
  "targetVariablePath": "funnel.incoming_text",
  "value": "{sys.message.text}"
}
```

Целевая переменная `funnel.incoming_text` должна быть заранее объявлена. Для полей, которые хранят JSON (`http_request.jsonBody`), используй JSON-aware resolution: самостоятельное выражение может вернуть JSON-тип, а смешанный шаблон интерполируется и затем разбирается как JSON. Проверяй, что результат остаётся валидным JSON.

## Имена переменных

В новых builder-выражениях используй только namespaced-пути:

- `sys.message.text`, `sys.client.name`, `sys.chat.channel`;
- `funnel.answer`, `funnel.api_response`;
- `env.API_URL`, `env.REGION_CODE`.

Некоторые старые примеры могут использовать `message.text` или `client.name`; для новых схем не копируй такие bare-пути. `sys.*` и `env.*` доступны только для чтения. Для изменения состояния используй `funnel.*` через поддерживаемый блок.

## Часто используемые функции

Ниже приведены функции, зарегистрированные в expression runtime проекта. Перед использованием неизвестной функции или аргумента сверяйся с capabilities/validator, а не с памятью или старым экспортом.

| Функция | Назначение | Пример |
|---|---|---|
| `lowercase(value)`, `uppercase(value)` | Изменить регистр | `{lowercase(sys.message.text)}` |
| `length(value)` | Длина строки | `{length(sys.message.text)}` |
| `replace(value, search, replace)` | Замена подстроки | `{replace(sys.message.text, "hi", "hello")}` |
| `regex_replace(value, pattern, replacement)` | Замена по PCRE | `{regex_replace(sys.message.text, '#^/start\\s+#', '')}` |
| `random(min, max)`, `floor(value)` | Числовые операции | `{random(1, 5)}` |
| `join(array, separator)`, `count(array)` | Обработка массива | `{join(sys.message.media.urls, ", ")}` |
| `first(array, fallback)`, `last(array, fallback)`, `slice(array, offset, limit)` | Выбор элементов массива | `{first(funnel.answers, "нет ответа")}` |
| `pluck(array, key)`, `unique(array)`, `contains(array, value)` | Обработка списка объектов/значений | `{contains(funnel.selected_tags, "vip")}` |
| `jsonDecode(value)` | Разобрать JSON-строку | `{jsonDecode(funnel.api_response).status}` |
| `DATE_FORMAT`, `ADD_DAYS`, `ADD_MONTHS`, `ADD_YEARS`, `ADD_HOURS`, `ADD_MINUTES`, `ADD_SECONDS` | Форматирование и арифметика дат | `{ADD_DAYS(sys.date, -5)}` |
| `base64(value)` | Кодирование строки | `{base64(sys.client.id)}` |

Аргументы и типы важны: например, `join` принимает массив, а `length` — строку. Для значений JSON выбери скалярное поле или индекс массива; не подставляй весь объект/массив в текст.

## Операторы и условия внутри expression

Expression Language поддерживает сравнения (`==`, `!=`, `>`, `>=`, `<`, `<=`), логические операторы (`and`, `or`, `not`), арифметику, `in`, `matches`, объединение строк через `~`, тернарное выражение и `??` для запасного значения.

```text
{sys.chat.channel == "telegram" ? "Telegram" : "Другой канал"}
{funnel.discount ?? 0}
{sys.message.text matches '#^/start\\s+#i'}
```

Для `matches` используй строку с delimiters и PCRE-паттерном. Без `^`/`$` поиск может совпасть внутри строки; добавляй границы, если нужна проверка всего текста. Неверный паттерн приводит к ошибке вычисления.

Условный оператор в expression возвращает значение. Он не создаёт графовую ветку. Для разных следующих шагов используй `condition` и `condition_else` с рёбрами; `condition.value` при этом остаётся строковым значением сравнения, а не expression.

## Ошибки и проверка

- Не используй `{}` или пустые placeholders.
- Не записывай результат в `sys.*`/`env.*`.
- Для `condition.variablePath` передавай путь без `{}`.
- Для `http_request.jsonBody` проверь валидность JSON после подстановки.
- `automation_schema_update` и `automation_enable` не заменяют проверку expression на реальном безопасном тестовом запуске. Не отправляй сообщение тестовому клиенту без просьбы пользователя.
