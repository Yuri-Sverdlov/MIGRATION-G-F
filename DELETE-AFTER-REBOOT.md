# DELETE-AFTER-REBOOT — оригиналы на G: для удаления

> Создано: 2026-08-25 (архитектор).
> Копии на F: проверены (src=dst). Удалять **только после перезагрузки Windows**,
> если папка занята процессом (Hermes, Ollama, Explorer).

## Как удалять

1. Перезагрузить Windows.
2. **Не запускать** терминал кодера / Hermes до удаления.
3. Проводник -> каждую папку -> **Shift+Delete** (мимо Корзины).
4. После удаления всех — написать архитектору «удалил после reboot».

## Список (10 папок)

| # | Путь на G: | Партия | Файлов (эталон) | Копия на F: |
|---|------------|--------|-----------------|-------------|
| 1 | `G:\AI\OllamaModels` | 1 | 56 | `F:\OllamaModels` |
| 2 | `G:\AI\for Google Drive` | 2a | 5 | `F:\for Google Drive` |
| 3 | `G:\AI\GGUF_Models` | 2a | 6 | `F:\GGUF_Models` (+ уникальные в `F:\OLLAMA-LMSTUDIO\GGUF_Models`) |
| 4 | `G:\AI\COMFYUI-RUNPOD` | 2a | 46 | `F:\COMFYUI-RUNPOD` |
| 5 | `G:\AI\_MY_PROGRAMMING_4` | 2b | 767 | `F:\_MY_PROGRAMMING_4` |
| 6 | `G:\AI\_MY_PROGRAMMING_2` | 2b | 4776 | `F:\_MY_PROGRAMMING_2` |
| 7 | `G:\AI\_MY_PROGRAMMING` | 4 | 101 019 | `F:\_MY_PROGRAMMING` |
| 8 | `G:\AI\COMFYUI-SHARED` | 5 | 221 | `F:\COMFYUI-SHARED` |
| 9 | `G:\AI\COMFYUI_PORTABLE` | 5 | 57 472 | `F:\COMFYUI_PORTABLE` |
| 10 | `G:\AI\COMFY_UI` | 5 | 43 294 | `F:\COMFY_UI` |

## Уже удалены (ничего не делать)

MonteCarloOptimizationKimi, OBSIDIAN-KNOWLEDGE-BASE, TOMATO-2-TIMERS, COURSES,
OLLAMA-LMSTUDIO, _NEW-PROJECT-TEMPLAT, MODELS-PROPERTY, _MY_PROGRAMMING_IVRIT,
COMFYUI-SYNCHRONIZATION, MODEL-SORT.

## Не удалять

| Путь | Причина |
|------|---------|
| `G:\AI\MIGRATION-G-F` | репозиторий проекта миграции |
| `G:\AI\_MY_PROGRAMMING_3` | ещё не мигрирован |
| `G:\AI\*` (файлы в корне) | финальная партия |

## Проверка после удаления

```
Test-Path "G:\AI\_MY_PROGRAMMING"   # -> False
Test-Path "F:\_MY_PROGRAMMING"    # -> True
```

Повторить для каждой из 10 папок.
