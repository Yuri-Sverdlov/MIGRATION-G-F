# CONTEXT.md — MIGRATION-G-F

**Суть.** Пошаговый перенос всего каталога `G:\AI\` на диск **F:** (в корень `F:\`).
Диск F: — песочница/рабочий том. Переносим партиями, аккуратно.

**Роли.**
- **Консультант** (Opus 4.8) — стратегия, разбор сложного, приёмка спорного. Дорогой, по минимуму.
- **Архитектор** (Composer, чат Cursor) — планы, пишет `tasks/TASK.md`, проверяет `tasks/REPORT.md`.
- **Кодер** (терминал) — выполняет задание.

**Стек/среда.** Windows, PowerShell 7 (`pwsh`), `robocopy`, `git`. Общие правила — `G:\AI\DEV-NOTES.md`. Защита от деструктива — `DEV-NOTES.md` §15.

**Метод миграции (для каждой папки):**
1. Копия через `robocopy` (гиганты — в фоне).
2. Проверка: число файлов src = dst.
3. Починка путей: жёсткие `G:\...` в скриптах/конфигах + внешние настройки приложений.
4. Удаление оригинала на G: — только после проверки.

**Текущий фокус.** Партии 1, 2a, 2b, 4, **5 (ComfyUI)** **закрыты**. Хвост: **10 оригиналов на G:** после reboot (`DELETE-AFTER-REBOOT.md`). Следующее: `_MY_PROGRAMMING_3` или финал.

---

## Закрытые партии

**Партия 1 (TASK-01):** MonteCarloOptimizationKimi, OBSIDIAN-KNOWLEDGE-BASE, TOMATO-2-TIMERS, COURSES, OLLAMA-LMSTUDIO, OllamaModels.

**Партия 2a (TASK-02):** _NEW-PROJECT-TEMPLAT, for Google Drive, GGUF_Models, MODELS-PROPERTY, _MY_PROGRAMMING_IVRIT, COMFYUI-SYNCHRONIZATION, COMFYUI-RUNPOD, MODEL-SORT.

**Партия 2b (TASK-03):** _MY_PROGRAMMING_4, _MY_PROGRAMMING_2.

**Партия 4 (TASK-04):** _MY_PROGRAMMING (~101k файлов).

**Партия 5 (TASK-05):** COMFYUI-SHARED (221), COMFYUI_PORTABLE (57472), COMFY_UI (43294).

---

## Очередь миграции (ещё на G:\AI\)

### Следующая
- `_MY_PROGRAMMING_3` (~83 505 файлов)

### Финал
- Корневые файлы `G:\AI\*` (часть уже на F:)
- Репозиторий `MIGRATION-G-F` — переносить последним

---

## Удаление после reboot

**Список:** `DELETE-AFTER-REBOOT.md` (10 папок, Shift+Delete).

**Документация:** `LESSONS-LEARNED.md` (грабли и решения), репозиторий на GitHub.
