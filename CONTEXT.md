# CONTEXT.md — MIGRATION-G-F

**Суть.** Пошаговый перенос всего каталога `G:\AI\` на диск **F:** (в корень `F:\`).
Диск F: — песочница/рабочий том. Переносим партиями, аккуратно.

**Роли.**
- **Консультант** (Opus 4.8) — стратегия, разбор сложного, приёмка спорного. Дорогой, по минимуму.
- **Архитектор** (Composer, чат Cursor) — планы, пишет `tasks/TASK.md`, проверяет `tasks/REPORT.md`.
- **Кодер** (терминал) — выполняет задание.

**Стек/среда.** Windows, PowerShell 7 (`pwsh`), `robocopy`, `git`. Общие правила — `F:\DEV-NOTES.md`. Защита от деструктива — `DEV-NOTES.md` §15.

**Метод миграции (для каждой папки):**
1. Копия через `robocopy` (гиганты — в фоне).
2. Проверка: число файлов src = dst.
3. Починка путей: жёсткие `G:\...` в скриптах/конфигах + внешние настройки приложений.
4. Удаление оригинала на G: — только после проверки.

**Текущий фокус.** **Миграция завершена (2026-09-10, TASK-08).** Рабочий репозиторий: `F:\MIGRATION-G-F`. На G:\ остались дубликаты — удалить пользователем (см. `DELETE-AFTER-REBOOT.md` §TASK-08).

---

## Закрытые партии

**Партия 1 (TASK-01):** MonteCarloOptimizationKimi, OBSIDIAN-KNOWLEDGE-BASE, TOMATO-2-TIMERS, COURSES, OLLAMA-LMSTUDIO, OllamaModels.

**Партия 2a (TASK-02):** _NEW-PROJECT-TEMPLAT, for Google Drive, GGUF_Models, MODELS-PROPERTY, _MY_PROGRAMMING_IVRIT, COMFYUI-SYNCHRONIZATION, COMFYUI-RUNPOD, MODEL-SORT.

**Партия 2b (TASK-03):** _MY_PROGRAMMING_4, _MY_PROGRAMMING_2.

**Партия 4 (TASK-04):** _MY_PROGRAMMING (~101k файлов).

**Партия 5 (TASK-05):** COMFYUI-SHARED (221), COMFYUI_PORTABLE (57472), COMFY_UI (43294).

**TASK-06 (2026-09-10):** удаление 10 оригиналов на G: (7 кодер + 3 пользователь ранее).

**TASK-07 (2026-09-10):** _MY_PROGRAMMING_3 (83 755 файлов) -> F:\.

**TASK-08 (2026-09-10):** финал — удаление G:\ `_MY_PROGRAMMING_3` (содержимое), корневые файлы -> F:\, repo -> `F:\MIGRATION-G-F`, path-fix, git push.

---

## Очередь миграции

**Пусто.** Все партии перенесены. Остатки на G: — только дубликаты для ручного удаления.

---

## DELETE-AFTER-REBOOT

**Закрыто (2026-09-10).** Все 10 оригиналов удалены. История: `DELETE-AFTER-REBOOT.md`.

**Документация:** `LESSONS-LEARNED.md`, репозиторий на GitHub.
