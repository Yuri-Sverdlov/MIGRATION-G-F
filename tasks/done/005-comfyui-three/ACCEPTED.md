# ACCEPTED — TASK-05 (3 ComfyUI-папки)

**Дата приёмки:** 2026-08-25
**Архитектор:** Composer (Cursor)

## Вердикт

**Принято с оговоркой:** кодер выполнил перенос корректно; архитектор доделал 1 обратную ссылку.

## Независимая проверка

- robocopy exit 1 для всех трёх; match=True:
  - COMFYUI-SHARED: **221 = 221**
  - COMFYUI_PORTABLE: **57 472 = 57 472**
  - COMFY_UI: **43 294 = 43 294**
- `config.py`: SHARED_ROOT, WORKING_COMFYUI, SCAN_ROOTS -> `F:\` (G:\Video, G:\DiskU, S:\ без изменений).
- `extra_model_paths.yaml` (SHARED + PORTABLE): `F:/COMFYUI-SHARED/models/`.
- `extra_models_config.yaml` (COMFY_UI, 2 копии): `F:\COMFY_UI`.
- `.claude\settings.local.json` (PORTABLE): `F:/COMFYUI_PORTABLE`, `F:/COMFYUI-SYNCHRONIZATION`.
- `install_path.txt`: уже `F:\COMFYUI_PORTABLE`.
- Контрольный скан по TASK (4 паттерна, 3 папки): **0 hits**.
- Оригиналы на G: **на месте** (не удалены).

## Доделано архитектором

- `F:\_MY_PROGRAMMING_2\ComfyUI\extra_model_paths.yaml`: `G:/AI/COMFYUI-SHARED` -> `F:/COMFYUI-SHARED` (рабочий ComfyUI, вне scope TASK-05, но критично для запуска).

## Не блокирует (заметка)

- `F:\COMFYUI-SHARED\_reports\*.json` — кэш inventory с JSON-escaped путями `G:\\AI\\...`; перегенерируется при следующем запуске inventory.

## Удаление оригиналов

Добавлены в `DELETE-AFTER-REBOOT.md` (п. 8–10). Удалять после reboot, Shift+Delete.

## Итог

ComfyUI-стек на F: готов по путям. Оригиналы `G:\AI\COMFYUI-SHARED`, `G:\AI\COMFYUI_PORTABLE`, `G:\AI\COMFY_UI` — удалить после reboot.
