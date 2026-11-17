# Builder-блоки: интеграции

Общие правила схемы — в [builder.md](../../builder.md). Выражения — в [Expression Language](../expression-language.md), пути и типы переменных — в [переменных](../variables.md).

У каждого блока обязательны `id` и `type`. Для HTTP, моделей и базы знаний сохраняй только известные ID и реальные значения; не выдумывай ресурсные UUID.

## `http_request`

Поля: `method`, `url`, `headers[]`, `queryParams[]`, `bodyType`, `jsonBody`, `rawBody`, `formData[]`, `urlEncodedBody[]`, `outputVariablePath`.

- `method`: `get`, `post`, `put`, `patch`, `delete`; URL указывается отдельно.
- `headers`, `queryParams`, `formData`, `urlEncodedBody` содержат `{id, key, value}`. Для активной записи нужен непустой `key`; пустые строки редактора игнорируются.
- `bodyType`: `none`, `form_data`, `x_www_form_urlencoded`, `json`, `raw`.
- При `none` тело не задаётся. При `json`/`raw` соответствующее поле (`jsonBody`/`rawBody`) должно быть непустым. При `form_data`/`x_www_form_urlencoded` нужен хотя бы один заполненный элемент соответствующего массива.
- URL, заголовки, параметры и тело могут содержать поддерживаемые expressions; для `jsonBody` сохраняй валидный JSON с безопасно интерполируемыми шаблонами.
- `outputVariablePath` необязателен. Если нужно сохранить тело ответа, укажи существующую строковую переменную `funnel.*`; поле не хранит произвольный тип ответа.

Не превращай блок в случайный внешний вызов для «проверки» схемы. HTTP-блок выполняет реальный запрос при запуске.

## `knowledge_base_search`

Поля: `knowledgeBaseId`, `query`, `scoreThreshold`, `topK`, `outputVariablePath`.

- База знаний должна существовать **в том же пространстве**, что и автоматизация. Инструмента `knowledge_bases_list` в GChat MCP нет: используй ID из схемы либо получи его у пользователя.
- `query` — непустой текстовый шаблон; выражение разрешается перед поиском и не должно стать пустым.
- `topK` — целое от 1 до 20; `scoreThreshold` — число от 0 до 1.
- Результат записывается в существующую строковую переменную `funnel.*` через обязательный `outputVariablePath`.

## `ai_router`

Поля: `modelId`, `inputVariablePath`, `instruction`, `minChoiceProbability`, `routes[]`, `outputVariablePath`.

- Нужны существующая модель, читаемый namespaced-путь входа и непустая инструкция. Справочника моделей в GChat MCP нет.
- `routes[]` содержит `{id, key, name, prompt}`. Требуется от 1 до 20 маршрутов. Ключ уникален, длиной 1–64 символа, только буквы, цифры, `-` и `_`; `__fallback` зарезервирован.
- Backend добавляет выход `__fallback`; его handle — `block-port:ai_router:<block.id>:__fallback`. Соедини fallback, если сценарий должен обрабатывать неуверенную классификацию.
- `minChoiceProbability` — `null` либо число от 0 до 1.
- `outputVariablePath` необязателен; если задан, это существующая строковая переменная `funnel.*` для выбранного ключа.
- Блок классифицирует и создаёт ветку, но сам не отвечает клиенту.

## `llm`

Поля: `modelId`, `inputVariablePath`, `messageWindow`, `generationMode`, `humanMessages`, `messages[]`, `outputVariablePath`.

- Нужны существующая модель и читаемый namespaced-путь `inputVariablePath`. Справочника моделей в MCP нет.
- `messages[]` содержит `{id, role, content}`. Первым идёт непустое сообщение `system`; остальные роли — `user` или `assistant`; других `system` быть не должно. Максимум 50 сообщений, каждое с непустым `content`.
- `messageWindow` имеет `{enabled, count, includeMedia}`. При `enabled: true` `count` должен быть от 1 до 20; `includeMedia` — boolean.
- При `generationMode: "single_text"` результат записывается в существующую строковую переменную `funnel.*` через `outputVariablePath`.
- При `generationMode: "human_messages"` передай `humanMessages: {minCount, maxCount}` с целыми границами 2–5 и `minCount <= maxCount`; `outputVariablePath` не используется как для `single_text`.

Параметры модели и фактический runtime могут иметь дополнительные ограничения провайдера; не заполняй `modelId` по догадке.
