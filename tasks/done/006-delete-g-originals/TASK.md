# TASK-06: удаление 7 оставшихся оригиналов на G:

> Задание: архитектор -> кодер. Отчёт пиши в `tasks/REPORT.md`.
> Правила среды: `AGENTS.md` (читай первым).
> `CONTEXT.md` не нужен — всё существенное ниже.

## Контекст

Миграция G:\ -> F:\ закрыта по копированию (партии 1–5). Остался хвост:
удалить **оригиналы** на G:, копии на F: уже проверены.

После reboot пользователь удалил **3 из 10** папок вручную (Shift+Delete):
`OllamaModels`, `COMFYUI-SHARED`, `COMFYUI_PORTABLE` — **OK, не трогать**.

**7 папок не удаляются** — Проводник: «Неверная функция MS-DOS» и др.
Причина: **8 файлов `nul`** (зарезервированные имена Windows) в `_MY_PROGRAMMING*`.
Копии на F: на месте — удалять G: **безопасно**.

**Не трогать:** `G:\AI\MIGRATION-G-F`, `G:\AI\_MY_PROGRAMMING_3`, файлы в корне `G:\AI\*`.

| # | Путь G: | Файлов на G: | Копия F: | Примечание |
|---|---------|-------------:|----------|------------|
| 1 | `G:\AI\_MY_PROGRAMMING_4` | ~4 | `F:\_MY_PROGRAMMING_4` (1051) | почти пуста, 4× `nul` |
| 2 | `G:\AI\_MY_PROGRAMMING_2` | ~1 | `F:\_MY_PROGRAMMING_2` (5315) | почти пуста, 1× `nul` |
| 3 | `G:\AI\for Google Drive` | 5 | `F:\for Google Drive` (5) | пробел в имени |
| 4 | `G:\AI\GGUF_Models` | 6 | `F:\GGUF_Models` (6) | мелкая |
| 5 | `G:\AI\COMFYUI-RUNPOD` | 49 | `F:\COMFYUI-RUNPOD` (49) | мелкая |
| 6 | `G:\AI\_MY_PROGRAMMING` | ~104 537 | `F:\_MY_PROGRAMMING` (104 541) | гигант, 3× `nul` |
| 7 | `G:\AI\COMFY_UI` | ~43 297 | `F:\COMFY_UI` (43297) | гигант |

Порядок удаления: **1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7** (от простого к тяжёлому).

---

## Правила среды (для этого задания)

- Оболочка: **PowerShell 7** (`pwsh`). Вывод в консоль — только **ASCII** (`->`, `<=`).
- В `REPORT.md` вставляй **полный текст вывода** каждой команды (DEV-NOTES §14:
  пустой вывод + код 0 — не доказательство успеха).
- **Не использовать** `Format-Table` / `Format-List` — хук ложно блокирует «Format»
  (DEV-NOTES §15, AGENTS.md).
- Пути с пробелами (`for Google Drive`) — только `-LiteralPath` и префикс `\\?\`.
- **Запрещено хуком:** `Remove-Item -Recurse`, `Remove-Item -Force` на деревьях,
  голый `del` без `\\?\`.
- **Разрешено для `nul`/`con`/`aux`/`prn`:** только `cmd /c del "\\?\полный_путь"`.
- **Разрешено для деревьев:** robocopy /MIR из пустой папки + `cmd /c rmdir /s /q "\\?\..."`.
- Сабагенты Hermes **не использовать**.

---

## Шаги

### 1. Проверка F: (обязательна, до любого удаления)

Выполни и вставь **полный вывод** в REPORT.md:

```
$fTargets = @(
  'F:\for Google Drive',
  'F:\GGUF_Models',
  'F:\COMFYUI-RUNPOD',
  'F:\_MY_PROGRAMMING_4',
  'F:\_MY_PROGRAMMING_2',
  'F:\_MY_PROGRAMMING',
  'F:\COMFY_UI'
)
foreach ($t in $fTargets) {
  $exists = Test-Path -LiteralPath $t
  $files = if ($exists) { (Get-ChildItem -LiteralPath $t -Recurse -File -Force -ErrorAction SilentlyContinue).Count } else { 0 }
  "F: OK=$exists files=$files | $t"
}
```

Ожидание: все 7 строк с `OK=True`. Если MISSING — **стоп**, удаление не начинать.

### 2. Удалить файлы `nul` (сначала!)

`Remove-Item` и обычный `del` **не работают**. Только:

```
cmd /c del "\\?\ПОЛНЫЙ_ПУТЬ"
```

Известные 8 файлов (удалять только если `Test-Path -LiteralPath` -> True):

```
G:\AI\_MY_PROGRAMMING\DB-WRITING-SYSTEM\nul
G:\AI\_MY_PROGRAMMING\DB-WRITING-SYSTEM\projects\nul
G:\AI\_MY_PROGRAMMING\DB-WRITING-SYSTEM\testing\nul
G:\AI\_MY_PROGRAMMING_2\COMFYUI-API-CHAT\nul
G:\AI\_MY_PROGRAMMING_4\AUTOMATION-LOWCONTENT-BOOK\nul
G:\AI\_MY_PROGRAMMING_4\DB-WRITING-SYSTEM\nul
G:\AI\_MY_PROGRAMMING_4\DB-WRITING-SYSTEM\projects\nul
G:\AI\_MY_PROGRAMMING_4\DB-WRITING-SYSTEM\testing\nul
```

Дополнительный поиск пропусков:

```powershell
$names = @('nul','con','aux','prn')
foreach ($root in @('G:\AI\_MY_PROGRAMMING','G:\AI\_MY_PROGRAMMING_2','G:\AI\_MY_PROGRAMMING_4','G:\AI\COMFY_UI','G:\AI\COMFYUI-RUNPOD','G:\AI\GGUF_Models')) {
  if (-not (Test-Path -LiteralPath $root)) { continue }
  Get-ChildItem -LiteralPath $root -Recurse -Force -ErrorAction SilentlyContinue |
    Where-Object { -not $_.PSIsContainer -and ($names -contains $_.Name.ToLower()) } |
    ForEach-Object {
      cmd /c del "\\?\$($_.FullName)"
      "deleted reserved: $($_.FullName) gone=$(-not (Test-Path -LiteralPath $_.FullName))"
    }
}
```

Вставь в отчёт вывод **каждого** `cmd /c del` и проверку `gone=True`.

### 3. Удаление 7 папок (robocopy /MIR + rmdir)

```powershell
$empty = 'C:\Users\Yuri\AppData\Local\Temp\empty_mirror_gf'
New-Item -ItemType Directory -Path $empty -Force | Out-Null

function Remove-TreeViaMirror {
  param([string]$Target)
  if (-not (Test-Path -LiteralPath $Target)) {
    "SKIP missing: $Target"
    return
  }
  robocopy $empty $Target /MIR /R:1 /W:1 /NFL /NDL /NJH /NJS
  $rc = $LASTEXITCODE
  cmd /c rmdir /s /q "\\?\$Target"
  $gone = -not (Test-Path -LiteralPath $Target)
  "Target=$Target gone=$gone robocopy_exit=$rc"
}

$targets = @(
  'G:\AI\_MY_PROGRAMMING_4',
  'G:\AI\_MY_PROGRAMMING_2',
  'G:\AI\for Google Drive',
  'G:\AI\GGUF_Models',
  'G:\AI\COMFYUI-RUNPOD',
  'G:\AI\_MY_PROGRAMMING',
  'G:\AI\COMFY_UI'
)
foreach ($t in $targets) { Remove-TreeViaMirror $t }
```

- Папки **1–5** — последовательно.
- Папки **6–7** (гиганты) — тот же метод; robocopy можно **в фоне**, но `rmdir`
  только **после** завершения robocopy для этой папки.
- Robocopy exit **0–7** = успех.

Если папка не удалилась — в отчёт: точный текст ошибки + вывод
`Get-ChildItem -LiteralPath $Target -Recurse -Force`.

### 4. Проверка после удаления

Выполни и вставь **полный вывод**:

```
$gTargets = @(
  'G:\AI\for Google Drive',
  'G:\AI\GGUF_Models',
  'G:\AI\COMFYUI-RUNPOD',
  'G:\AI\_MY_PROGRAMMING_4',
  'G:\AI\_MY_PROGRAMMING_2',
  'G:\AI\_MY_PROGRAMMING',
  'G:\AI\COMFY_UI'
)
foreach ($t in $gTargets) {
  "G: gone=$(-not (Test-Path -LiteralPath $t)) | $t"
}
Test-Path -LiteralPath 'F:\_MY_PROGRAMMING'
Test-Path -LiteralPath 'F:\COMFY_UI'
Test-Path -LiteralPath 'G:\AI\MIGRATION-G-F'
Test-Path -LiteralPath 'G:\AI\_MY_PROGRAMMING_3'
```

Ожидание: все 7 `gone=True`; F: и защищённые папки — `True`.

### 5. Обновить DELETE-AFTER-REBOOT.md

Если все 7 удалены — в секции «Статус» проставить **удалена (TASK-06)** для п. 2–7.
Не менять строки уже удалённых пользователем (п. 1, 8, 9).

### 6. Git commit + push

Только если шаг 4 успешен (7/7 gone=True):

```
git add DELETE-AFTER-REBOOT.md tasks/REPORT.md
git commit -m "TASK-06: deleted 7 G: originals after nul cleanup"
git push origin main
```

В отчёт — вывод `git status` после push и `git rev-parse HEAD`.

### 7. Заполни REPORT.md

В отчёте должно быть:

- Полный вывод шага 1 (F: OK, 7 строк).
- Список удалённых `nul`/`con`/`aux`/`prn` с `gone=True` для каждого.
- Для **каждой** из 7 папок: `robocopy_exit`, `gone=True/False`.
- Полный вывод шага 4 (проверка G: + F: + защищённые папки).
- Git: commit hash + push OK (или причина, если не пушил).
- Итоговый вердикт: «7/7 удалены, F: цел» — либо описание проблем.

---

## Критерии приёмки

- [ ] F: копии проверены до удаления (7/7 OK=True) — вывод приложен.
- [ ] Все `nul`/`con`/`aux`/`prn` удалены через `cmd /c del "\\?\..."` — вывод приложен.
- [ ] 7 папок на G: не существуют (`gone=True`) — вывод приложен.
- [ ] F: копии на месте после операции — вывод приложен.
- [ ] `MIGRATION-G-F`, `_MY_PROGRAMMING_3`, корневые файлы не тронуты.
- [ ] `DELETE-AFTER-REBOOT.md` обновлён (если 7/7 OK).
- [ ] `REPORT.md` с **полным текстом** команд (не пересказ, не «тихий зелёный»).
- [ ] Git push на `origin/main` (если 7/7 OK).

## Чего НЕ делать

- Не использовать `Remove-Item -Recurse` / `-Force` на деревьях.
- Не использовать `Format-Table` / `Format-List`.
- Не трогать F: (удалять только G:).
- Не использовать сабагентов Hermes.
- Не удалять `_MY_PROGRAMMING_3`, `MIGRATION-G-F`, корневые файлы `G:\AI\*`.
- Не начинать удаление, если шаг 1 показал MISSING на F:.
