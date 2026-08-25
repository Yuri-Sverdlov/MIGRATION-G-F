# TASK-02: партия 2a — перенос 8 мелких папок G: -> F:

> Задание: архитектор -> кодер. Отчёт пиши в `tasks/REPORT.md`.
> Правила среды: `AGENTS.md` (читай первым).

## Контекст

Партия 1 завершена (архив: `tasks/done/001-batch1-verify/`).
Переносим следующие 8 папок из `G:\AI\` в корень `F:\`.
Все папки мелкие (1–110 файлов), robocopy без `/MT:16` — синхронно.

**Оригиналы на G: не удалять** — это делает пользователь после приёмки архитектором.

## Список папок

| # | SRC (G:) | DST (F:) | Файлов (эталон) |
|---|----------|----------|-----------------|
| 1 | `G:\AI\_NEW-PROJECT-TEMPLAT` | `F:\_NEW-PROJECT-TEMPLAT` | 1 |
| 2 | `G:\AI\for Google Drive` | `F:\for Google Drive` | 5 |
| 3 | `G:\AI\GGUF_Models` | `F:\GGUF_Models` | 6 |
| 4 | `G:\AI\MODELS-PROPERTY` | `F:\MODELS-PROPERTY` | 7 |
| 5 | `G:\AI\_MY_PROGRAMMING_IVRIT` | `F:\_MY_PROGRAMMING_IVRIT` | 22 |
| 6 | `G:\AI\COMFYUI-SYNCHRONIZATION` | `F:\COMFYUI-SYNCHRONIZATION` | 31 |
| 7 | `G:\AI\COMFYUI-RUNPOD` | `F:\COMFYUI-RUNPOD` | 46 |
| 8 | `G:\AI\MODEL-SORT` | `F:\MODEL-SORT` | 110 |

## Шаг 1. Копирование (robocopy)

Для **каждой** папки из таблицы:

```
robocopy "G:\AI\<ИМЯ>" "F:\<ИМЯ>" /E /COPY:DAT /R:1 /W:1
```

- Код выхода robocopy 0–7 = успех (0 = ничего не копировалось, 1+ = скопировано).
- Код 8+ = ошибка, остановись и опиши в REPORT.md.
- Если DST уже существует — robocopy дозапишет/обновит (это норма).

## Шаг 2. Проверка (обязательна)

После каждой пары SRC/DST — число файлов должно совпасть:

```
$src = (Get-ChildItem "G:\AI\<ИМЯ>" -Recurse -File).Count
$dst = (Get-ChildItem "F:\<ИМЯ>" -Recurse -File).Count
"G:\AI\<ИМЯ> -> src=$src dst=$dst match=$($src -eq $dst)"
```

**Без match=True перенос этой папки не засчитывать.**

Сводную таблицу всех 8 пар вставь в REPORT.md.

## Шаг 3. Скан жёстких путей G:\ в копиях на F:

Для каждой скопированной папки на F: найди файлы с жёсткими путями `G:\`
(только текстовые: `.py`, `.ps1`, `.bat`, `.json`, `.env`, `.md`, `.txt`, `.yml`, `.yaml`, `.cfg`, `.ini`):

```
Get-ChildItem "F:\<ИМЯ>" -Recurse -File |
  Where-Object { $_.Extension -match '\.(py|ps1|bat|json|env|md|txt|yml|yaml|cfg|ini)$' } |
  ForEach-Object {
    if (Select-String -Path $_.FullName -Pattern 'G:\\' -SimpleMatch -Quiet) {
      $_.FullName
    }
  }
```

**Ложные совпадения:** строки кода вроде `"...string:\n"` могут дать `g:\` — проверяй глазами, не правь шум.

### Известные пути — что менять, что нет

**Менять** (папка уже на F:):
- `G:\AI\COMFYUI-RUNPOD\...` -> `F:\COMFYUI-RUNPOD\...`
- `G:\AI\COMFYUI-SYNCHRONIZATION\...` -> `F:\COMFYUI-SYNCHRONIZATION\...`
- `G:\AI\_MY_PROGRAMMING_IVRIT\...` -> `F:\_MY_PROGRAMMING_IVRIT\...`
- `G:\AI\COMFYUI-SYNCHRONIZATION\install_path.txt`: `G:\AI\COMFYUI_PORTABLE` -> `F:\COMFYUI_PORTABLE`
  (ComfyUI Portable ещё не перенесён, но целевой путь уже F:)

**НЕ менять** (папка ещё на G:, перенос позже):
- `G:\AI\_MY_PROGRAMMING\...` — гигант, партия 3+
- `G:\AI\COMFYUI_PORTABLE` — в markdown/README можно оставить как есть, кроме `install_path.txt`
- `G:\_My_Programming-2\...` — другой диск/путь, не трогать

**Особый случай `_MY_PROGRAMMING_IVRIT`:**
- `translate_bridge.py`: путь `G:\AI\_MY_PROGRAMMING\HERMES-AGENT\.openrouter_key` — **оставить**,
  `_MY_PROGRAMMING` ещё не мигрирован.

### `.venv`

Если найдёшь `.venv` — **не копировать и не править**, пересоздадим при запуске проекта.

## Шаг 4. Особый случай: GGUF_Models

`G:\AI\GGUF_Models` — **legacy-папка** (6 файлов .gguf).
Рабочий каталог LM Studio уже на `F:\OLLAMA-LMSTUDIO\GGUF_Models` (партия 1).

После копирования в `F:\GGUF_Models` — сравни имена файлов с
`F:\OLLAMA-LMSTUDIO\GGUF_Models` (рекурсивно). В REPORT.md таблица:

| Файл в F:\GGUF_Models | Есть в LM Studio dir? | Действие |
|---|---|---|
| ... | да/нет | дубликат / уникальный |

**Уникальные** файлы (flux, pixelwave и т.п.) — **скопируй** в
`F:\OLLAMA-LMSTUDIO\GGUF_Models\` (если там ещё нет). Не удаляй `F:\GGUF_Models`.

## Шаг 5. Заполни REPORT.md

Структура отчёта:

1. **Robocopy** — код выхода для каждой из 8 папок.
2. **Сверка файлов** — таблица src/dst/match для всех 8 пар.
3. **Скан G:\** — список файлов с путями + что исправлено (до/после).
4. **GGUF_Models** — таблица дубликатов/уникальных + куда скопированы уникальные.
5. **Итоговый вердикт** — все 8 папок на F:, match=True, пути починены.

## Критерии приёмки

- [ ] Все 8 папок скопированы на F: (robocopy exit 0–7).
- [ ] src=dst для всех 8 пар (match=True).
- [ ] Жёсткие `G:\AI\<эта_папка>` заменены на `F:\<эта_папка>` в копиях.
- [ ] `install_path.txt` указывает `F:\COMFYUI_PORTABLE`.
- [ ] GGUF: уникальные файлы перенесены в LM Studio dir, дубликаты задокументированы.
- [ ] REPORT.md с полным выводом команд (не пересказ, не «тихий зелёный»).
- [ ] Оригиналы на G: **не удалены**.

## Чего НЕ делать

- Не удалять ничего на G: (хук заблокирует).
- Не использовать `Format-Table` / `Format-List` (хук ложно блокирует).
- Не трогать папки вне списка 8.
- Не менять git-remote URL (это не пути диска).
- Не менять пути на папки, которые ещё не мигрированы (`_MY_PROGRAMMING` и т.п.).
