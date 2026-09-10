# DELETE-AFTER-REBOOT — оригиналы на G: для удаления

> Создано: 2026-08-25 (архитектор). Обновлено: 2026-09-10 (TASK-06, финал).
> **Все 10 оригиналов удалены.** Файл оставлен для истории.

## Финальный статус (2026-09-10, TASK-06)

| # | Путь на G: | Статус |
|---|------------|--------|
| 1 | `G:\AI\OllamaModels` | **удалена** пользователем |
| 2 | `G:\AI\for Google Drive` | **удалена (TASK-06)** |
| 3 | `G:\AI\GGUF_Models` | **удалена (TASK-06)** |
| 4 | `G:\AI\COMFYUI-RUNPOD` | **удалена (TASK-06)** |
| 5 | `G:\AI\_MY_PROGRAMMING_4` | **удалена (TASK-06)** |
| 6 | `G:\AI\_MY_PROGRAMMING_2` | **удалена (TASK-06)** |
| 7 | `G:\AI\_MY_PROGRAMMING` | **удалена (TASK-06)** |
| 8 | `G:\AI\COMFYUI-SHARED` | **удалена** пользователем |
| 9 | `G:\AI\COMFYUI_PORTABLE` | **удалена** пользователем |
| 10 | `G:\AI\COMFY_UI` | **удалена (TASK-06)** |

**История:** файлы `nul` удалены через `cmd /c del "\\?\..."`. Папки обнулены через
`robocopy /MIR`. Три папки не удалялись сразу (GoogleDriveFS держал handle) — удалены
после остановки процесса GoogleDriveFS (pid 32348).

## Уже удалены (ничего не делать)

MonteCarloOptimizationKimi, OBSIDIAN-KNOWLEDGE-BASE, TOMATO-2-TIMERS, COURSES,
OLLAMA-LMSTUDIO, _NEW-PROJECT-TEMPLAT, MODELS-PROPERTY, _MY_PROGRAMMING_IVRIT,
COMFYUI-SYNCHRONIZATION, MODEL-SORT.

## Не удалять

| Путь | Причина |
|------|---------|
| `F:\MIGRATION-G-F` | репозиторий проекта миграции |
| `G:\AI\_MY_PROGRAMMING_3` | ещё не мигрирован |
| `G:\AI\*` (файлы в корне) | финальная партия |

## Проверка

Все 7 папок TASK-06 проверены: `Test-Path` -> False для G:, True для F:.
Все 3 папки, удалённые пользователем ранее, также отсутствуют на G:.
