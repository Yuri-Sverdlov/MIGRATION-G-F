# TASK-03: партия 2b — перенос _MY_PROGRAMMING_4 и _MY_PROGRAMMING_2

> Задание: архитектор -> кодер. Отчёт пиши в `tasks/REPORT.md`.
> Правила среды: `AGENTS.md` (читай первым).

## Контекст

Партии 1 и 2a завершены (архивы: `tasks/done/001-*`, `tasks/done/002-*`).
Переносим 2 средние папки из `G:\AI\` в корень `F:\`.

**Оригиналы на G: не удалять** — пользователь удалит после приёмки архитектором
(часть старых оригиналов ждёт перезагрузки Windows — не твоя задача).

## Список папок

| # | SRC (G:) | DST (F:) | Файлов (эталон) |
|---|----------|----------|-----------------|
| 1 | `G:\AI\_MY_PROGRAMMING_4` | `F:\_MY_PROGRAMMING_4` | 767 |
| 2 | `G:\AI\_MY_PROGRAMMING_2` | `F:\_MY_PROGRAMMING_2` | 4776 |

`_MY_PROGRAMMING_2` крупнее — robocopy с `/MT:16`, можно в фоне.

## Шаг 1. Копирование (robocopy)

**Папка 1** (синхронно):

```
robocopy "G:\AI\_MY_PROGRAMMING_4" "F:\_MY_PROGRAMMING_4" /E /COPY:DAT /R:1 /W:1
```

**Папка 2** (фон, `/MT:16`):

```
robocopy "G:\AI\_MY_PROGRAMMING_2" "F:\_MY_PROGRAMMING_2" /E /COPY:DAT /R:1 /W:1 /MT:16
```

- Exit code 0–7 = успех; 8+ = ошибка, стоп + REPORT.
- Не трогать другие папки на G:.

## Шаг 2. Проверка (обязательна)

```
$pairs = @(
  @{G='G:\AI\_MY_PROGRAMMING_4'; F='F:\_MY_PROGRAMMING_4'},
  @{G='G:\AI\_MY_PROGRAMMING_2'; F='F:\_MY_PROGRAMMING_2'}
)
foreach ($p in $pairs) {
  $src = (Get-ChildItem $p.G -Recurse -File).Count
  $dst = (Get-ChildItem $p.F -Recurse -File).Count
  "$($p.G) -> src=$src dst=$dst match=$($src -eq $dst)"
}
```

**Без match=True перенос не засчитывать.** Вывод в REPORT.md.

## Шаг 3. Скан и починка путей G:\ в копиях на F:

Скан текстовых файлов (`.py`, `.ps1`, `.bat`, `.json`, `.env`, `.md`, `.txt`, `.yml`, `.yaml`, `.cfg`, `.ini`) на `G:\` в обеих папках на F:.

### Правила замены

**Менять — операционные пути этих папок:**

| Было | Стало |
|------|-------|
| `G:\AI\_MY_PROGRAMMING_4\...` | `F:\_MY_PROGRAMMING_4\...` |
| `G:\AI\_MY_PROGRAMMING_2\...` | `F:\_MY_PROGRAMMING_2\...` |
| `G:\AI\DEV-NOTES.md` | `F:\DEV-NOTES.md` |
| `G:\AI\setup-new-pc.ps1` | `F:\setup-new-pc.ps1` |
| `G:\AI\OllamaModels` | `F:\OllamaModels` |
| `G:\AI\COMFYUI-SHARED` | `F:\COMFYUI-SHARED` |
| `G:\AI\COMFYUI_PORTABLE` (корневой, не вложенный) | `F:\COMFYUI_PORTABLE` |

**Важно — два разных COMFYUI_PORTABLE:**

- `G:\AI\COMFYUI_PORTABLE\...` — **корневая** гигант-папка (ещё не мигрирована) -> `F:\COMFYUI_PORTABLE\...`
- `G:\AI\_MY_PROGRAMMING_2\COMFYUI_PORTABLE\...` — **вложенная** копия внутри _MY_PROGRAMMING_2 -> `F:\_MY_PROGRAMMING_2\COMFYUI_PORTABLE\...`

Не путать! В bat/py с `_MY_PROGRAMMING_2\COMFYUI_PORTABLE` меняй на `F:\_MY_PROGRAMMING_2\COMFYUI_PORTABLE`.

**Исправить явный баг в _MY_PROGRAMMING_4:**

- `DB-WRITING-SYSTEM\run-gui.bat` и `start-gui.bat` сейчас указывают на
  `G:\AI\_MY_PROGRAMMING\DB-WRITING-SYSTEM` — проект уже в _MY_PROGRAMMING_4.
  Заменить на `F:\_MY_PROGRAMMING_4\DB-WRITING-SYSTEM`.

**НЕ менять:**

- `G:\AI\_MY_PROGRAMMING\...` (кроме run-gui.bat выше) — гигант, партия 3+
- `G:\_My_Programming\...`, `G:\_My_Programming-2\...` — другие пути
- `G:\___Planning_Life_Sphere\...`, `G:\Projects_Life_planning\...` — другие проекты
- Исторические строки «**Откуда:** `G:\AI\_MY_PROGRAMMING\...`» в README (миграционная история)
- Файлы `session-*.json` — логи сессий Hermes, не трогать
- git-remote URL (`https://github.com/...`)

**Ложные совпадения:** `"...string:\n"` даёт `g:\` — проверяй глазами.

### `.venv`

Не копировать отдельно, не править — пересоздадим при запуске.

## Шаг 4. Приоритет файлов для починки

Сначала исполняемые/конфиги (обязательно):

- `*.bat`, `*.ps1`, `*.py` с путями
- `AGENTS.md`, `CONTEXT.md`, `tasks/TASK.md` в подпроектах
- `extra_model_paths.yaml`, `.env`, `*.yaml` конфиги ComfyUI

Потом документация (`README.md`, `QUICK_START.md`) — только строки с рабочими
путями (`cd G:\AI\...`, абсолютные пути в инструкциях), не исторические «Откуда/Куда».

## Шаг 5. Заполни REPORT.md

1. Robocopy exit codes (обе папки).
2. Сверка src/dst (обе пары, match=True).
3. Список файлов с G:\ (скан) + таблица исправлений (файл, было, стало).
4. Список намеренно оставленных G:\ (с пояснением).
5. Итоговый вердикт.

## Критерии приёмки

- [ ] Обе папки на F:, robocopy exit 0–7.
- [ ] match=True для обеих пар.
- [ ] Операционные пути `_MY_PROGRAMMING_4` / `_MY_PROGRAMMING_2` -> F:.
- [ ] COMFYUI-SHARED / COMFYUI_PORTABLE (корневой) -> F:\ (целевые пути).
- [ ] run-gui.bat / start-gui.bat -> `F:\_MY_PROGRAMMING_4\DB-WRITING-SYSTEM`.
- [ ] REPORT.md с полным выводом (не пересказ).
- [ ] Оригиналы на G: не удалены.

## Чего НЕ делать

- Не удалять на G:.
- Не использовать `Format-Table` / `Format-List`.
- Не переносить `COMFY_UI`, `COMFYUI-SHARED` (корневые) — они в отдельной партии позже.
- Не править `session-*.json`.
