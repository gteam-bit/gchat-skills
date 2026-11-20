# Каталог MCP-инструментов GChat

Сервер может показывать только те инструменты, на которые у MCP-токена есть право. Неизвестные аргументы на верхнем уровне запрещены; вложенные объекты следуют схеме своего поля. `schema` у `automation_schema_update` — открытый объект: backend проверяет структуру графа и отдельные поля с разной глубиной, а часть условий и значений проверяется только при runtime. Для точного payload используй [builder-справочники](builder.md); успешное сохранение/включение не доказывает, что все ветки исполнятся. Необязательные аргументы отмечены `?`; UUID передавай строкой.

Успешный ответ инструмента содержит `status` и `data`. Ошибки авторизации, валидации и отсутствующих ресурсов возвращаются как ошибки MCP. `readOnlyHint: false` означает побочный эффект — отправку сообщения, изменение данных или состояния.

## Пагинация каталога MCP

MCP-метод `tools/list` может вернуть только часть опубликованных инструментов и поле `nextCursor`. Начни с `params: {}`, затем вызывай `tools/list` с `params: {"cursor": "<nextCursor>"}` и повторяй, пока `nextCursor` не исчезнет. Ищи нужный инструмент среди результатов **всех** страниц. Не считай отсутствие на первой странице признаком отсутствия инструмента или права на него.

Это пагинация каталога инструментов MCP. Она не связана с аргументами `page`, `limit` и `cursor` отдельных инструментов ниже. Если клиент показывает сохранённый список инструментов и после полного обхода серверного каталога новый инструмент в нём не появился, обнови подключение клиента.

## Теги

| Инструмент | Аргументы | Права / эффект |
|---|---|---|
| `tags_list` | `search?`, `page?` (по умолчанию 1), `limit?` (1–500, по умолчанию 15) | `TAGS_VIEW_ANY`, чтение |
| `tag_get` | `id` | `TAGS_VIEW_ANY`, чтение |
| `tag_create` | `name`, `key?`, `color?` | `TAGS_CREATE`, запись |
| `tag_update` | `id`, `name?`, `key?`, `color?` | `TAGS_UPDATE`, запись |
| `tag_delete` | `id` | `TAGS_DELETE`, удаление |

`color` — строка до 7 символов; идентификатор тега для автоматизации бери из `tags_list` или `tag_get`.

## Пользовательские поля организации

| Инструмент | Аргументы | Права / эффект |
|---|---|---|
| `custom_fields_list` | `search?`, `page?` (1), `limit?` (1–500, 15) | `CUSTOM_FIELDS_VIEW_ANY`, чтение |
| `custom_field_get` | `id` | `CUSTOM_FIELDS_VIEW_ANY`, чтение |
| `custom_field_create` | `name`, `key`, `type`, `options?`, `default_value_text?`, `default_value_json?`, `is_visible?`, `is_editable?` | `CUSTOM_FIELDS_CREATE`, запись |
| `custom_field_update` | `id`, `name?`, `key?`, `type?`, `options?`, `default_value_text?`, `default_value_json?`, `is_visible?`, `is_editable?` | `CUSTOM_FIELDS_UPDATE`, запись |
| `custom_field_delete` | `id` | `CUSTOM_FIELDS_DELETE`, удаление |

Значения `type` бери из enum схемы инструмента. `options` и `default_value_json` — списки строк.

## Чаты и клиенты

### Чаты

| Инструмент | Аргументы | Права / эффект |
|---|---|---|
| `chats_list` | `status?`, `statuses?`, `tab?`, `search?`, `integration_id?`, `integration_ids?`, `staff_profile_ids?`, `staff_group_ids?`, `created_at_from?`, `created_at_to?`, `last_message_at_from?`, `last_message_at_to?`, `last_incoming_at_from?`, `last_incoming_at_to?`, `client_tag_ids?`, `custom_field_id?`, `custom_field_value?`, `integration_service?`, `exclude_integration_service?`, `read_without_reply?`, `without_reply?`, `cursor?`, `limit?` (1–100, 25) | `CHATS_VIEW_ANY`, чтение |
| `chat_get` | `chat` | `CHATS_VIEW_ANY`, чтение |
| `chat_messages_list` | `chat`, `cursor?`, `search?` (1–255 символов), `limit?` (1–100) | `CHATS_VIEW_ANY`, чтение |
| `chat_assignees_list` | `chat`, `search?`, `page?` (1), `limit?` (1–50) | `CHATS_ASSIGN`, чтение |
| `chat_transfer_groups_list` | `chat`, `search?`, `page?` (1), `limit?` (1–50) | `CHATS_ASSIGN`, чтение |
| `chat_send_text` | `chat`, `text` (1–4096 символов) | `CHATS_VIEW_ANY`, отправляет сообщение |
| `chat_status_update` | `chat`, `status` | `CHATS_SET_MANUAL_STATUS`, меняет статус |
| `chat_assign` | `chat`, `staff_profile_id?`, `group_id?`, `comment?` | `CHATS_ASSIGN`, назначает чат |
| `chat_mark_read` | `chat` | `CHATS_MARK_READ`, меняет отметку прочтения |

`chats_list.search` ищет по имени и телефону клиента, UUID клиента, внешнему ID клиента, UUID и внешнему ID чата. Страница содержит краткий объект `client` с `id`, `name`, `phone`, тегами. Список использует `cursor`, а не `page`.

### Клиенты

| Инструмент | Аргументы | Права / эффект |
|---|---|---|
| `client_get` | `client` | `CLIENTS_MANAGE_DETAILS`, чтение комментария, тегов и значений полей |
| `client_tags_list` | `client` | `CLIENTS_MANAGE_TAGS`, чтение |
| `client_tag_add` | `client`, `tag_id` | `CLIENTS_MANAGE_TAGS`, меняет теги |
| `client_tag_remove` | `client`, `tag_id` | `CLIENTS_MANAGE_TAGS`, меняет теги |
| `client_comment_update` | `client`, `comment` (строка или `null`) | `CLIENTS_MANAGE_DETAILS`, запись |
| `client_custom_fields_update` | `client`, `values` (непустой список объектов `{custom_field_id, value_text?, value_json?}`) | `CLIENTS_MANAGE_DETAILS`, запись |

Отдельного `clients_list` сейчас нет. Ищи клиента по имени через `chats_list`, затем проверяй выбранный UUID через `client_get`.

## Пространства и builder-автоматизации

| Инструмент | Аргументы | Права / эффект |
|---|---|---|
| `automation_spaces_list` | `search?`, `page?` (1), `limit?` (1–100, 15) | `AUTOMATION_SPACES_VIEW_ANY`, чтение |
| `automation_space_get` | `space_id` | `AUTOMATION_SPACES_VIEW_ANY`, чтение |
| `automations_list` | `space_id`, `search?`, `page?` (1), `limit?` (1–100, 15) | `AUTOMATION_FUNNELS_VIEW_ANY`, чтение |
| `automation_get` | `space_id`, `automation_id` | `AUTOMATION_FUNNELS_VIEW_ANY`, метаданные без схемы блоков |
| `automation_create` | `space_id`, `name`, `trigger_type?`, `trigger_settings?` | `AUTOMATION_FUNNELS_CREATE`, создаёт выключенной |
| `automation_update` | `space_id`, `automation_id`, `name`, `trigger_type?`, `trigger_settings?` | `AUTOMATION_FUNNELS_UPDATE`, полная замена метаданных, `enabled` сохраняется |
| `automation_delete` | `space_id`, `automation_id` | `AUTOMATION_FUNNELS_DELETE`, удаление |
| `automation_enable` | `space_id`, `automation_id` | `AUTOMATION_FUNNELS_UPDATE`, проверяет сохранённую схему и включает |
| `automation_disable` | `space_id`, `automation_id` | `AUTOMATION_FUNNELS_UPDATE`, выключает |
| `automation_schema_get` | `space_id`, `automation_id` | `AUTOMATION_FUNNELS_VIEW_ANY`, чтение схемы и `schema_version` |
| `automation_schema_update` | `space_id`, `automation_id`, `expected_schema_version` (целое ≥1), `schema` (полный объект) | `AUTOMATION_FUNNELS_UPDATE`, полная замена схемы |
| `automation_schema_update_from_upload` | `space_id`, `automation_id`, `expected_schema_version` (целое ≥1), `upload_id` | `AUTOMATION_FUNNELS_UPDATE`, полная замена схемы из временного JSON-файла |

Типы `trigger_type`: `message.received`, `client.tag.changed`, `client.tag.added`, `client.tag.removed`, `client.custom_field.changed`. Для входящего сообщения в `trigger_settings.mode` допустимы `any_message` и `first_client_message`. Для триггеров тегов передавай `tag_ids` — список из 1–20 UUID тегов этой организации. Для изменения пользовательского поля используй `target_id` с UUID поля. Для тегов поддерживается и `client.tag.changed`, и более точные `client.tag.added`/`client.tag.removed`.

Оба инструмента обновления заменяют схему целиком. `schema.version` — версия формата схемы; `expected_schema_version` — номер сохранённой ревизии. Для небольшой схемы вызови `automation_schema_get` и передай его ревизию в `automation_schema_update`; для крупной используй [загрузку JSON-файлом](schema-file-upload.md) и `automation_schema_update_from_upload`. При конфликте проверь текущую ревизию и полную схему перед повтором. После обновления включай автоматизацию отдельным `automation_enable`.

Когда схема не помещается в один MCP-вызов, используй [загрузку JSON-файлом](schema-file-upload.md). Временный файл не является медиафайлом схемы; `mediaFileId` внутри самого JSON обрабатываются как раньше.

## Базы знаний

Базы знаний принадлежат builder-пространству. `space_id` бери из `automation_spaces_list` и проверяй через `automation_space_get`; `knowledge_base_id` — из `knowledge_bases_list`. Организация задаётся MCP-токеном.

| Инструмент | Аргументы | Права / эффект |
|---|---|---|
| `knowledge_bases_list` | `space_id`, `search?` (до 255 символов), `page?` (1), `limit?` (1–100, 15) | `KNOWLEDGE_BASES_VIEW_ANY`, чтение списка и счётчиков ресурсов/фрагментов |
| `knowledge_base_get` | `space_id`, `knowledge_base_id` | `KNOWLEDGE_BASES_VIEW_ANY`, чтение метаданных базы и счётчиков |
| `knowledge_base_search` | `space_id`, `knowledge_base_id`, `query` (1–200 символов), `top_k?` (1–20, 5) | `KNOWLEDGE_BASES_VIEW_ANY`, поиск фрагментов |
| `knowledge_base_resource_create` | `space_id`, `knowledge_base_id`, `name` (1–255 символов), `content` (1–100 000 символов) | `KNOWLEDGE_BASE_RESOURCES_CREATE`, создаёт текстовый Markdown-ресурс и ставит его в очередь обработки |

`knowledge_base_search` возвращает фрагменты с `resource_id`, `resource_name`, `chunk_id`, `content`, `score` и доступными данными источника. Это найденные материалы, а не готовый ответ на вопрос. После `knowledge_base_resource_create` ответ содержит ID ресурса и статус `queued`: обработка и индексация происходят позже. Эти четыре инструмента не показывают статус отдельного ресурса; пустой поиск сразу после создания не доказывает, что текст не добавлен. Создание не принимает готовый документ или URL. Повторный вызов создания может добавить второй ресурс, поэтому не повторяй его автоматически после неопределённого результата.

## Медиафайлы

Бинарную загрузку выполняй multipart-запросом на `POST /mcp/media-uploads` с тем же MCP Bearer; это HTTP endpoint, а не вызов `tools/call`. Форматы, поля загрузки, права и порядок получения временной ссылки описаны в [справочнике по медиафайлам](media-files.md).

| Инструмент | Аргументы | Права / эффект |
|---|---|---|
| `media_list` | `query?` (до 255 символов), `page?` (1), `per_page?` (1–100, 50) | `AUTOMATION_FUNNELS_VIEW_ANY`, поиск файлов по имени и временные ссылки |
| `media_get` | `media_file_id` | `AUTOMATION_FUNNELS_VIEW_ANY`, метаданные и временная ссылка |
| `media_get_many` | `media_file_ids` (список 1–100 UUID) | `AUTOMATION_FUNNELS_VIEW_ANY`, метаданные и временные ссылки |

## Логи и трейсы автоматизаций

Все инструменты ниже требуют `space_id` и `automation_id`, работают в области организации MCP-токена и доступны с `AUTOMATION_FUNNELS_VIEW_ANY`.

| Инструмент | Дополнительные аргументы | Назначение |
|---|---|---|
| `automation_logs_conversations_list` | `search?`, `status?`, `page?` (1), `limit?` (1–100, 15) | Список conversation-сессий; `search` ищет в том числе по UUID клиента |
| `automation_logs_conversation_get` | `conversation_id` | Подробности сессии и последнего запуска |
| `automation_logs_conversation_outline` | `conversation_id`, `cursor?` (до 512 символов), `limit?` (1–500, 200), `direction?` (`asc`/`desc`, по умолчанию `asc`) | Постраничный обзор событий трейса |
| `automation_logs_trace_event_get` | `conversation_id`, `event_id` | Одно событие и связанные сообщения |
| `automation_logs_conversation_messages` | `conversation_id`, `cursor?`, `limit?` (1–500, 500), `direction?` (`asc`/`desc`), `anchor_message_id?` (целое ≥1) | Постраничные сообщения; `anchor_message_id` нельзя передавать вместе с `cursor` |
| `automation_logs_runs_list` | `search?`, `status?`, `page?` (1), `limit?` (1–100, 15) | Поиск запусков этой автоматизации |
| `automation_logs_run_trace` | `run_id` | Полный трейс одного запуска |

Допустимые статусы запуска: `running`, `waiting`, `completed`, `failed`, `cancelled`. Для поиска по клиенту используй список conversations с `search=<client UUID>`; не полагайся на поиск клиентского UUID через `automation_logs_runs_list`. Для длинной диагностики сначала читай outline страницами и загружай нужные события по `event_id`; полный `run_trace` запрашивай, когда действительно нужен весь запуск. `completed` означает завершённый запуск, но само по себе не доказывает бизнес-конверсию.
