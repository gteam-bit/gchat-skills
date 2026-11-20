# Медиафайлы для automation funnels

Используй этот workflow, когда нужно добавить изображение, голосовое сообщение или video note в builder-схему, найти уже загруженный файл или скачать его для проверки. MCP-токен определяет организацию; не передавай `organization_id`.

## Загрузка

Медиа загружается отдельным multipart HTTP endpoint, а не инструментом JSON-RPC `tools/call`. Отправь запрос на `/mcp/media-uploads` того же сервера с тем же MCP Bearer.

| Поле | Условие |
|---|---|
| `file` | Обязательный файл до 100 МиБ. Поддерживаются JPG/JPEG, PNG, MP4, OGG, OGA и OPUS. |
| `kind` | Необязательно: `default`, `voice` или `video_note`. Для `send_voice` укажи `voice`, для `send_video_note` — `video_note`, чтобы применился соответствующий pipeline подготовки. |
| `original_name` | Необязательное имя для поиска, до 255 символов. |

Например, клиент с shell может отправить файл через `curl`:

```sh
curl --fail-with-body --silent --show-error \
  --header "Authorization: Bearer ${GCHAT_MCP_TOKEN}" \
  --form "file=@${MEDIA_FILE}" \
  --form "kind=voice" \
  --form "original_name=welcome.ogg" \
  "${GCHAT_BASE_URL}/mcp/media-uploads"
```

Это пример HTTP-клиента, а не обязательный способ загрузки. Если `original_name` не передан, сервер использует исходное имя multipart-файла. Ответ содержит `id`, `original_name`, `mime_type`, `file_size`, доступные размеры/длительность и `upload_claim_id`; пути хранилища и `public_url` не возвращаются. Загрузка требует `MEDIA_AUTOMATION_UPLOAD`.

## Поиск и скачивание

- `media_list` ищет файлы по подстроке имени и возвращает страницу; по умолчанию `page=1`, `per_page=50`, максимум — 100 элементов на страницу.
- `media_get` получает один файл по `media_file_id`; `media_get_many` принимает от 1 до 100 UUID.
- Инструменты чтения требуют `AUTOMATION_FUNNELS_VIEW_ANY`. Доступ определяется upload claim или ссылкой на файл в схеме воронки текущей организации.
- Ответы `media_get`, `media_get_many` и `media_list` содержат метаданные и `download_url`. Ссылка действует 15 минут; скачивай по точному URL до `expires_at`. Для новой ссылки снова вызови `media_get` или `media_list`. Не сохраняй URL в схему builder.

## Использование в схеме

Сохраняй в builder-схеме ID файла, а не временную ссылку:

- `send_media_text` использует `mediaFileIds`;
- `send_voice` и `send_video_note` используют `mediaFileId`.

Перед записью проверь имя и тип по результату загрузки или `media_get`. При сохранении схемы сервер проверяет доступность каждого media ID для организации. Полная структура блоков описана в [блоках сообщений](builder/blocks/messages.md).
