# MIGRATION-G-F

Пошаговая миграция каталога `G:\AI\` на диск **F:** (в корень `F:\`), партиями, с проверкой и починкой путей.

**Репозиторий:** [github.com/Yuri-Sverdlov/MIGRATION-G-F](https://github.com/Yuri-Sverdlov/MIGRATION-G-F)

## Статус (2026-09-10)

| Закрыто | Папок | TASK |
|---------|------:|------|
| Партия 1 | 6 | TASK-01 |
| Партия 2a | 8 | TASK-02 |
| Партия 2b | 2 | TASK-03 |
| Партия 4 | 1 (~101k файлов) | TASK-04 |
| Партия 5 (ComfyUI) | 3 (~101k файлов) | TASK-05 |
| Удаление G: | 10 оригиналов | TASK-06 |
| Партия 6 | 1 (~84k файлов) | TASK-07 |
| Финал | корневые файлы + repo | TASK-08 |

**Миграция завершена (2026-09-10).** Рабочая копия: **`F:\MIGRATION-G-F`**.

**Остатки на G:\AI\** (дубликаты, Shift+Delete после reboot): пустая оболочка `_MY_PROGRAMMING_3`, `MIGRATION-G-F`, корневые файлы — см. [`DELETE-AFTER-REBOOT.md`](DELETE-AFTER-REBOOT.md) §TASK-08.

**DELETE-AFTER-REBOOT (партии 1–5):** закрыт (10/10, 2026-09-10).

## Как устроен проект

Трёхролевая схема:

- **Архитектор** (Cursor chat) — пишет `tasks/TASK.md`, принимает работу, ведёт документацию.
- **Кодер** (терминал / Hermes) — выполняет robocopy, path-fix, пишет `tasks/REPORT.md`.
- **Консультант** (Opus) — эскалация на сложных местах.

## Ключевые файлы

| Файл | Назначение |
|------|------------|
| [`AGENTS.md`](AGENTS.md) | Устав кодера: robocopy, безопасность, запреты |
| [`CONTEXT.md`](CONTEXT.md) | Текущий фокус и очередь миграции |
| [`LESSONS-LEARNED.md`](LESSONS-LEARNED.md) | **Грабли, решения, что учесть** |
| [`PROJECT_LOG.md`](PROJECT_LOG.md) | Хронология по дням |
| [`DELETE-AFTER-REBOOT.md`](DELETE-AFTER-REBOOT.md) | Список оригиналов G: для Shift+Delete |
| [`tasks/done/`](tasks/done/) | Архив принятых заданий (TASK + REPORT + ACCEPTED) |

## Метод (кратко)

```
robocopy G:\AI\FOLDER F:\FOLDER /E /COPY:DAT /R:1 /W:1 [/MT:16]
-> сверка числа файлов src = dst
-> замена G:\AI\... на F:\... в конфигах (все форматы путей!)
-> приёмка архитектором
-> удаление G: пользователем (не агентом)
```

Подробности и все грабли — в [`LESSONS-LEARNED.md`](LESSONS-LEARNED.md).

## Безопасность

Действуют хуки Cursor: агент **не может** массово удалять файлы (`Remove-Item -Recurse`, `format` и т.д.). Удаление больших оригиналов — только пользователь (Shift+Delete). См. `F:\DEV-NOTES.md` §15.
