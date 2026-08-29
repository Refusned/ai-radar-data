# Контракт выпуска AI Radar

Каноническая JSON Schema:

`https://raw.githubusercontent.com/Refusned/ai-radar-data/main/schema/ai-radar-run.schema.json`

## Manifest `latest.json`

- `schema_version`: всегда `1.0`.
- `current_snapshot`: относительный путь к текущему snapshot.
- `generated_at`: время формирования указанного snapshot.
- `status`: статус указанного snapshot.

## Верхний уровень snapshot

- `schema_version`: всегда `1.0`.
- `run_key`: `ai-radar:<scheduled_for с часовым поясом>`.
- `attempt_id`: уникальный идентификатор попытки.
- `generated_at`, `window_start`, `window_end`: ISO 8601.
- `status`: `complete`, `partial` или `failed`.
- `source_health[]`: состояние каждого обязательного канала.
- `news_items[0..5]`.
- `repositories[0..5]`.
- `daily_action`: один проверяемый эксперимент или `null`.
- `corrections[]`, `errors[]`.

## Идемпотентность

Один `run_key` соответствует одному выпуску. Повторная попытка не создает второй логический выпуск. Разрешен переход `partial` в `complete`, запрещен откат `complete` в `partial`.

Путь snapshot строить из `generated_at`, заменяя двоеточия на дефисы. Перед созданием проверить, что путь не существует. При конфликте прочитать существующий файл и сопоставить `run_key`.

## GitHub-метрики

Для каждого репозитория сохранять текущее число stars и forks, время снимка, `tracking_since`, дату последнего push, язык, лицензию и наблюдаемые сигналы. `delta_24h` остается `null`, пока нет двух собственных snapshots с известным интервалом.

## Запись

1. Создать snapshot.
2. Убедиться чтением, что он доступен и соответствует `run_key`.
3. Получить текущий SHA `latest.json`.
4. Обновить manifest `latest.json` полями `schema_version`, `current_snapshot`, `generated_at` и `status`, передав текущий SHA.
5. Перечитать manifest и snapshot.

Если шаги 3-5 не завершены, старый `latest.json` остается источником истины.
