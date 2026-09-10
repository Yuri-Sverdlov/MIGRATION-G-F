# TASK-08: финал миграции G:\AI\ -> F:\

> Задание: архитектор -> кодер. Отчёт пиши в `tasks/REPORT.md`.
> Правила среды: `AGENTS.md` (читай первым).
> `CONTEXT.md` не нужен — всё существенное ниже.

## Контекст

Партии 1–5, TASK-06, TASK-07 **закрыты**. На `G:\AI\` осталось:

| Что | Действие |
|-----|----------|
| `G:\AI\_MY_PROGRAMMING_3` | оригинал — **удалить** (копия на F: проверена TASK-07) |
| 35 корневых **файлов** | часть уже на `F:\` — **досинхронизировать** + пути |
| `F:\MIGRATION-G-F` | репозиторий проекта — **перенести последним** |

**Целевой корень:** `F:\`, не `F:\AI\`.

**Оригиналы на G: не удалять** (кроме `_MY_PROGRAMMING_3` по фазе A) — пользователь
удалит остальное после приёмки.

---

## Правила среды (для этого задания)

- Оболочка: **PowerShell 7** (`pwsh`). Вывод в консоль — только **ASCII**.
- В `REPORT.md` — **полный текст вывода** каждой команды (DEV-NOTES §14).
- **Не использовать** `Format-Table` / `Format-List`.
- **Запрещено хуком:** `Remove-Item -Recurse`, `Remove-Item -Force` на деревьях,
  голый `del` без `\\?\`.
- Удаление `_MY_PROGRAMMING_3`: метод из TASK-06 — robocopy /MIR + `cmd /c rmdir`.
  Если sharing violation — проверить **GoogleDriveFS**, остановить, повторить.
- Сабагенты Hermes **не использовать**.
- Кириллические имена файлов — `-LiteralPath`, не строкой в команде.

---

## Фаза A. Удаление `G:\AI\_MY_PROGRAMMING_3`

### A1. Проверка F: (до удаления)

```
$src = (Get-ChildItem "G:\AI\_MY_PROGRAMMING_3" -Recurse -File -Force -ErrorAction SilentlyContinue).Count
$dst = (Get-ChildItem "F:\_MY_PROGRAMMING_3" -Recurse -File -Force -ErrorAction SilentlyContinue).Count
"G:\AI\_MY_PROGRAMMING_3 -> src=$src dst=$dst match=$($src -eq $dst)"
```

Ожидание: **match=True** (~83 755). Без match — **стоп**, не удалять.

### A2. Удаление (robocopy /MIR + rmdir)

```powershell
$empty = 'C:\Users\Yuri\AppData\Local\Temp\empty_mirror_gf'
New-Item -ItemType Directory -Path $empty -Force | Out-Null
$target = 'G:\AI\_MY_PROGRAMMING_3'
robocopy $empty $target /MIR /R:1 /W:1 /NFL /NDL /NJH /NJS
cmd /c rmdir /s /q "\\?\$target"
"gone=$(-not (Test-Path -LiteralPath $target)) robocopy_exit=$LASTEXITCODE"
```

При exit 32 (sharing violation) — остановить GoogleDriveFS, повторить `rmdir`
(см. TASK-06 REPORT, LESSONS-LEARNED §2.7).

### A3. Проверка

```
Test-Path -LiteralPath 'G:\AI\_MY_PROGRAMMING_3'   # -> False
Test-Path -LiteralPath 'F:\_MY_PROGRAMMING_3'      # -> True
```

---

## Фаза B. Корневые файлы `G:\AI\*` -> `F:\`

### B1. Инвентаризация

Сравнить файлы **только в корне** (не папки):

```powershell
$gFiles = Get-ChildItem 'G:\AI' -File -Force
$fMap = @{}
Get-ChildItem 'F:\' -File -Force | ForEach-Object { $fMap[$_.Name] = $_.Length }
foreach ($f in $gFiles) {
  $onF = $fMap.ContainsKey($f.Name)
  $match = if ($onF) { $fMap[$f.Name] -eq $f.Length } else { $false }
  "name=$($f.Name) | onF=$onF | sizeMatch=$match"
}
```

**Пропустить:** `~$*` (временный lock Office).

**Скопировать на F:** файлы, которых нет на F: или размер на G: **новее**.
Использовать `Copy-Item -LiteralPath` (по одному) или:

```
robocopy "G:\AI" "F:\" /XO /R:1 /W:1 /FFT
  /XF "~$*"
```

`/XO` = только новее; не перезаписывать более свежие на F:.

На 2026-09-10 **точно отсутствуют на F:** (эталон архитектора):
- `DEV-NOTES-BACKLOG.md`
- `2-Модели сравнения разных, - для Гермес .docx`
- `модели  openrouter в Гермесе .jpg`
- `модели  openrouter в Гермесе2 .jpg`
- `Модели сравнения разных, - для Гермес .docx`

### B2. Проверка после копирования

Повторить B1 — все файлы (кроме `~$*`) должны быть `onF=True sizeMatch=True`.

---

## Фаза C. Пути в корневых операционных файлах на F:

Заменить `G:\AI\` -> `F:\` (все форматы) **только** в корне `F:\`:

| Файл | Что менять |
|------|------------|
| `F:\DEV-NOTES.md` | канонический путь `F:\DEV-NOTES.md` -> `F:\DEV-NOTES.md` |
| `F:\setup-new-pc.ps1` | ссылки на `F:\DEV-NOTES.md` |
| `F:\download_flux_full.py` | `G:\AI\COMFY_UI_MODELS\...` -> осмысленный путь на F: |
| `F:\download_sdxl.py` | то же (`G:\AI\COMFY_UI_MODELS\...`) |

**Примечание:** папки `COMFY_UI_MODELS` нет ни на G:, ни на F:. Заменить на
`F:\COMFYUI-SHARED\models\...` (или другой существующий путь на F:) и указать
выбор в REPORT.md.

**НЕ менять:** содержимое `.docx`, `.jpg`, `.mp4`, `.7z`, `.lnk` (бинарники).
`AI.lnk` — проверить target через отчёт (`$sh = (New-Object -ComObject WScript.Shell).CreateShortcut('F:\AI.lnk'); $sh.TargetPath`), не править автоматически.

Скрипт (корень F:\, только текстовые):

```powershell
$root = 'F:\'
$files = @('DEV-NOTES.md','setup-new-pc.ps1','download_flux_full.py','download_sdxl.py','fix_key.bat','CODING_Session-Report_Layers-2-3-Hardening.md','CODING_The-most-important-information.md')
$replacements = @(
  @{ Old = 'G:\\AI\\'; New = 'F:\\' },
  @{ Old = 'G:/AI/'; New = 'F:/' },
  @{ Old = 'G:\AI\'; New = 'F:\' },
  @{ Old = 'g:/AI/'; New = 'f:/' },
  @{ Old = 'g:\AI\'; New = 'f:\' }
)
$changed = 0
foreach ($name in $files) {
  $path = Join-Path $root $name
  if (-not (Test-Path -LiteralPath $path)) { continue }
  $text = [IO.File]::ReadAllText($path)
  $orig = $text
  foreach ($r in $replacements) { $text = $text.Replace($r.Old, $r.New) }
  if ($text -ne $orig) {
    [IO.File]::WriteAllText($path, $text, [Text.UTF8Encoding]::new($false))
    $changed++
    "Updated: $name"
  }
}
"Root files changed: $changed"
```

### C2. Контрольный скан (корень F:\)

```
$patterns = @('G:\AI\','G:/AI/','G:\\AI\\')
$files = @('DEV-NOTES.md','setup-new-pc.ps1','download_flux_full.py','download_sdxl.py')
$hits = 0
foreach ($name in $files) {
  $p = "F:\$name"
  if (-not (Test-Path -LiteralPath $p)) { continue }
  foreach ($pat in $patterns) {
    if (Select-String -Path $p -Pattern $pat -SimpleMatch -Quiet) { $hits++; "$name -> $pat" }
  }
}
"root operational hits=$hits"
```

Ожидание: **0 hits**.

---

## Фаза D. Перенос `MIGRATION-G-F` (последним)

### D1. Robocopy

```
robocopy "F:\MIGRATION-G-F" "F:\MIGRATION-G-F" /E /COPY:DAT /R:1 /W:1 /MT:8
```

Ждать завершения. Exit 0–7 = OK.

### D2. Проверка

```
$src = (Get-ChildItem "F:\MIGRATION-G-F" -Recurse -File -Force -ErrorAction SilentlyContinue).Count
$dst = (Get-ChildItem "F:\MIGRATION-G-F" -Recurse -File -Force -ErrorAction SilentlyContinue).Count
"MIGRATION-G-F -> src=$src dst=$dst match=$($src -eq $dst)"
```

### D3. Пути в `F:\MIGRATION-G-F`

Заменить `F:\MIGRATION-G-F` -> `F:\MIGRATION-G-F` и `F:\DEV-NOTES.md` -> `F:\DEV-NOTES.md`
в: `AGENTS.md`, `CLAUDE.md`, `HANDOFF-1.md`, `README.md` (если есть жёсткие пути).

**Git-remote URL не трогать.**

### D4. Git commit + push **из `F:\MIGRATION-G-F`**

```
Set-Location 'F:\MIGRATION-G-F'
git status
git add -A
git commit -m "TASK-08: final migration docs and F: paths"
git push origin main
```

В отчёт: `git rev-parse HEAD` после push.

---

## Фаза E. Финальная проверка

```
Test-Path -LiteralPath 'G:\AI\_MY_PROGRAMMING_3'      # False
Test-Path -LiteralPath 'F:\_MY_PROGRAMMING_3'         # True
Test-Path -LiteralPath 'F:\MIGRATION-G-F\tasks\TASK.md'
Test-Path -LiteralPath 'F:\MIGRATION-G-F'          # True (оригинал repo, не удалять)
```

---

## Шаг. Заполни REPORT.md

1. Фаза A: match до удаления, gone=True, F: цел.
2. Фаза B: таблица B1 до/после, список скопированных файлов.
3. Фаза C: число изменённых файлов, scan 0 hits, статус AI.lnk.
4. Фаза D: robocopy exit, match, commit hash, push OK.
5. Фаза E: все Test-Path.
6. Итоговый вердикт.

---

## Критерии приёмки

- [ ] `_MY_PROGRAMMING_3` на G: удалён; F: копия цела.
- [ ] Корневые файлы G:\AI синхронизированы с F:\ (кроме `~$*`).
- [ ] `DEV-NOTES.md`, `setup-new-pc.ps1`, download-скрипты — пути на F:\.
- [ ] `F:\MIGRATION-G-F` — полная копия repo, match=True.
- [ ] AGENTS.md / CLAUDE.md указывают на `F:\DEV-NOTES.md`.
- [ ] Git push с `F:\MIGRATION-G-F` успешен.
- [ ] REPORT.md с полным выводом команд.
- [ ] `F:\MIGRATION-G-F` и корневые файлы на G: **не удалены** (кроме `_MY_PROGRAMMING_3`).

## Чего НЕ делать

- Не удалять `F:\MIGRATION-G-F` и корневые файлы G:\ (кроме `_MY_PROGRAMMING_3`).
- Не использовать `Remove-Item -Recurse` / `-Force` на деревьях.
- Не трогать папки проектов на F:\ (массовый path-fix — не scope).
- Не править `.docx`/`.lnk`/`.7z` автоматически.
- Не использовать сабагентов.
