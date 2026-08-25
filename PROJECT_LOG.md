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
