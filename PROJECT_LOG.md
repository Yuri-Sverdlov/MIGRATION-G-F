# PROJECT_LOG — MIGRATION-G-F (append-only)

## 2026-08-25 — старт, партия 1 (консультант)
- Корневые файлы `G:\AI\*` уже скопированы пользователем в `F:\` (ранее).
- Скопированы + проверены (число файлов совпало) на F: (корень) 6 папок:
  MonteCarloOptimizationKimi, OBSIDIAN-KNOWLEDGE-BASE, TOMATO-2-TIMERS, COURSES,
  OLLAMA-LMSTUDIO (129 ГБ), OllamaModels (59 ГБ).
- Пути переключены на F::
  - `OLLAMA_MODELS` user = `F:\OLLAMA-LMSTUDIO\OllamaModels`; machine = `F:\OllamaModels`.
  - LM Studio `downloadsFolder` = `F:\OLLAMA-LMSTUDIO\GGUF_Models`.
- Оригиналы на G: ещё НЕ удалены. Осталось: проверка Ollama/LM Studio -> удаление 6 оригиналов.
- Замечен ложный триггер хука на `Format-Table` (принят за `format`). Зафиксировать в отчёте.

## 2026-08-25 — TASK-01 принят (архитектор)
- Кодер (Hermes/deepseek-v4-flash): проверка Ollama (17 моделей, тест OK) и LM Studio (13 GGUF на F:).
- Архитектор: независимая перепроверка — факты совпали, галлюцинаций нет.
- Пользователь удалил 5 из 6 оригиналов на G: (Shift+Delete).
- **`G:\AI\OllamaModels` не удалена** — занята процессом; удаление после перезагрузки Windows.
- Архив: `tasks/done/001-batch1-verify/` (TASK + REPORT + ACCEPTED).
- Следующий шаг: TASK-02, партия 2a (мелкие папки).

## 2026-08-25 — TASK-02 принят (архитектор)
- Кодер: 8 папок скопированы на F:, match=True; пути починены; 3 уникальных GGUF -> LM Studio dir.
- Архитектор: перепроверка OK; исправлен `F:\AI\COMFYUI-RUNPOD` -> `F:\COMFYUI-RUNPOD`.
- Пользователь удалил 5 из 8 оригиналов партии 2a.
- **Не удалены:** `for Google Drive`, `GGUF_Models`, `COMFYUI-RUNPOD` (+ `OllamaModels` из партии 1).
- Вероятная причина: процессы Hermes/кодера держат handle. Закрыть терминал или перезагрузка.
- Архив: `tasks/done/002-batch2a-small/`.
- **Отложено пользователем:** `COMFY_UI`, `COMFYUI-SHARED` — тяжёлые, перенос позже (зафиксировано в CONTEXT.md).
- Следующий шаг: TASK-03, партия 2b (`_MY_PROGRAMMING_4`, `_MY_PROGRAMMING_2`).

## 2026-08-25 — TASK-04 закрыт (архитектор)
- Кодер: robocopy OK, 101019=101019, 69 файлов путей (без сабагента).
- Архитектор: скан кодера неполный (пропущены G:/AI/ и G:\\\\AI\\\\); доделано 7 .py.
- Пользователь: удалил что смог; 7 папок на G: ждут reboot (см. DELETE-AFTER-REBOOT.md).
- Архив: `tasks/done/004-my-programming/`.
- Следующий шаг: reboot -> удалить 7 папок; TASK-05 (_MY_PROGRAMMING_3) или ComfyUI.

## 2026-08-25 — TASK-05 выдан (архитектор)
- Пользователь выбрал ComfyUI (3 папки) вместо `_MY_PROGRAMMING_3`.
- Задание: `tasks/TASK.md` — COMFYUI-SHARED (221), COMFYUI_PORTABLE (57472), COMFY_UI (43294).
- Порядок A->B->C; гиганты с `/MT:16` в фоне; path-fix без сабагента.
- `install_path.txt` на F: уже указывает `F:\COMFYUI_PORTABLE`.

## 2026-08-25 — TASK-05 принят (архитектор)
- Кодер: robocopy OK, match=True (221, 57472, 43294); пути в 3 папках починены.
- Архитектор: перепроверка OK; доделан `F:\_MY_PROGRAMMING_2\ComfyUI\extra_model_paths.yaml`.
- Оригиналы на G: не удалены; добавлены в DELETE-AFTER-REBOOT (итого 10 папок).
- Архив: `tasks/done/005-comfyui-three/`.
- Следующий шаг: reboot -> удалить 10 папок; TASK-06 (_MY_PROGRAMMING_3).

## 2026-08-25 — документация + GitHub (архитектор)
- Создан `LESSONS-LEARNED.md` — грабли, решения, правила на будущее.
- Создан `README.md`; обновлены `AGENTS.md`, `CONTEXT.md`.
- Push в https://github.com/Yuri-Sverdlov/MIGRATION-G-F (архив tasks/done/ 001–005).

## 2026-09-10 — TASK-06 принят (архитектор)
- Кодер: 7/7 оригиналов на G: удалены; 8× `nul` через `cmd /c del "\\?\..."`; 3 папки после stop GoogleDriveFS.
- F: цел (7/7); защищённые папки не тронуты. Commit push: `107963b` (в REPORT кодера ошибочно `44471a1`).
- Архив: `tasks/done/006-delete-g-originals/`. DELETE-AFTER-REBOOT закрыт (10/10).
- Следующий шаг: TASK-07 (`_MY_PROGRAMMING_3`).

## 2026-09-10 — TASK-07 принят (архитектор)
- Кодер: robocopy OK, 83755=83755 (robocopy); пути 34 файла; scan 0 hits (неполный).
- Архитектор: match=True (83755); доделан `config.json` (JSON-escaped ref_audio).
- Оригинал G: не удалён. Архив: `tasks/done/007-my-programming-3/`.
- Следующий шаг: удалить G:\ `_MY_PROGRAMMING_3`; TASK-08 финал.

## 2026-09-10 — TASK-08 выдан (архитектор)
- Финал: удаление G:\ `_MY_PROGRAMMING_3`, корневые файлы G:\AI -> F:\, repo на F:\MIGRATION-G-F.
- 5 файлов отсутствуют на F: (DEV-NOTES-BACKLOG + 4 docx/jpg); ~29 уже совпадают по размеру.
