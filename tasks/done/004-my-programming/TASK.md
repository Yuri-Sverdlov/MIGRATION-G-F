# TASK-04: перенос _MY_PROGRAMMING (гигант, ~101k файлов)

> Задание: архитектор -> кодер. Отчёт пиши в `tasks/REPORT.md`.
> Правила среды: `AGENTS.md` (читай первым).

## Контекст

Партии 1, 2a, 2b закрыты. Переносим **одну** гигант-папку:

| SRC (G:) | DST (F:) | Файлов (эталон) |
|----------|----------|-----------------|
| `G:\AI\_MY_PROGRAMMING` | `F:\_MY_PROGRAMMING` | **101 019** |

**Оригинал на G: не удалять** — пользователь удалит после приёмки (вероятно после reboot).

**Важно:** сабагенты Hermes **не использовать** (credits exhausted на TASK-03).
Массовую замену путей делать **самому** — PowerShell-скриптом (шаблон ниже).

---

## Шаг 1. Robocopy (фон, /MT:16)

```
robocopy "G:\AI\_MY_PROGRAMMING" "F:\_MY_PROGRAMMING" /E /COPY:DAT /R:1 /W:1 /MT:16
```

- Запускать **в фоне** — ~101k файлов, может занять десятки минут.
- Exit code 0–7 = успех; 8+ = ошибка, стоп + REPORT.
- Ждать завершения перед шагом 2.

---

## Шаг 2. Проверка (обязательна)

```
$src = (Get-ChildItem "G:\AI\_MY_PROGRAMMING" -Recurse -File).Count
$dst = (Get-ChildItem "F:\_MY_PROGRAMMING" -Recurse -File).Count
"G:\AI\_MY_PROGRAMMING -> src=$src dst=$dst match=$($src -eq $dst)"
```

**Без match=True перенос не засчитывать.** Вывод в REPORT.md.

---

## Шаг 3. Массовая замена путей на F:

### 3a. Внутри `F:\_MY_PROGRAMMING`

Скрипт (запустить как есть, проверить вывод `Changed files: N`):

```powershell
$root = 'F:\_MY_PROGRAMMING'
$extPattern = '\.(py|ps1|bat|json|env|md|txt|yml|yaml|cfg|ini)$'
$skipName = '^session-.*\.json$'

$replacements = @(
  @{ Old = 'G:\\AI\\_MY_PROGRAMMING\\';   New = 'F:\\_MY_PROGRAMMING\\' },
  @{ Old = 'G:\\AI\\_MY_PROGRAMMING_4\\'; New = 'F:\\_MY_PROGRAMMING_4\\' },
  @{ Old = 'G:\\AI\\_MY_PROGRAMMING_2\\'; New = 'F:\\_MY_PROGRAMMING_2\\' },
  @{ Old = 'G:\\AI\\_MY_PROGRAMMING_IVRIT\\'; New = 'F:\\_MY_PROGRAMMING_IVRIT\\' },
  @{ Old = 'G:\\AI\\DEV-NOTES.md';       New = 'F:\\DEV-NOTES.md' },
  @{ Old = 'G:\\AI\\setup-new-pc.ps1';   New = 'F:\\setup-new-pc.ps1' },
  @{ Old = 'G:\\AI\\OllamaModels';       New = 'F:\\OllamaModels' },
  @{ Old = 'G:\\AI\\COMFYUI-SHARED';     New = 'F:\\COMFYUI-SHARED' },
  @{ Old = 'G:\\AI\\COMFYUI_PORTABLE';    New = 'F:\\COMFYUI_PORTABLE' },
  @{ Old = 'G:\AI\_MY_PROGRAMMING\';    New = 'F:\_MY_PROGRAMMING\' },
  @{ Old = 'G:\AI\_MY_PROGRAMMING_4\';   New = 'F:\_MY_PROGRAMMING_4\' },
  @{ Old = 'G:\AI\_MY_PROGRAMMING_2\';   New = 'F:\_MY_PROGRAMMING_2\' },
  @{ Old = 'G:\AI\_MY_PROGRAMMING_IVRIT\'; New = 'F:\_MY_PROGRAMMING_IVRIT\' },
  @{ Old = 'G:\AI\_MY_PROGRAMMING';       New = 'F:\_MY_PROGRAMMING' },
  @{ Old = 'G:\AI\DEV-NOTES.md';         New = 'F:\DEV-NOTES.md' },
  @{ Old = 'G:\AI\setup-new-pc.ps1';     New = 'F:\setup-new-pc.ps1' },
  @{ Old = 'G:\AI\OllamaModels';         New = 'F:\OllamaModels' },
  @{ Old = 'G:\AI\COMFYUI-SHARED';       New = 'F:\COMFYUI-SHARED' },
  @{ Old = 'G:\AI\COMFYUI_PORTABLE';     New = 'F:\COMFYUI_PORTABLE' },
  @{ Old = 'g:\AI\_MY_PROGRAMMING\';     New = 'f:\_MY_PROGRAMMING\' },
  @{ Old = 'g:\AI\_MY_PROGRAMMING';      New = 'f:\_MY_PROGRAMMING' }
)

$changed = @()
Get-ChildItem $root -Recurse -File -ErrorAction SilentlyContinue |
  Where-Object { $_.Extension -match $extPattern -and $_.Name -notmatch $skipName } |
  ForEach-Object {
    $path = $_.FullName
    $text = [IO.File]::ReadAllText($path)
    $orig = $text
    foreach ($r in $replacements) { $text = $text.Replace($r.Old, $r.New) }
    if ($text -ne $orig) {
      [IO.File]::WriteAllText($path, $text, [Text.UTF8Encoding]::new($false))
      $changed += $path
    }
  }
Write-Output "Changed files: $($changed.Count)"
```

### 3b. Обратные ссылки (уже на F:)

Папки `_MY_PROGRAMMING_4` и `_MY_PROGRAMMING_IVRIT` уже на F: и **ссылаются**
на `G:\AI\_MY_PROGRAMMING\...` (ключи, скрипты). После копирования — прогнать
**тот же скрипт** с `$root = 'F:\_MY_PROGRAMMING_4'` и `$root = 'F:\_MY_PROGRAMMING_IVRIT'`
(только замены `G:\AI\_MY_PROGRAMMING` -> `F:\_MY_PROGRAMMING`).

### Правила

**Менять:** все операционные пути `G:\AI\_MY_PROGRAMMING\...` -> `F:\...`

**НЕ менять:**
- `G:\_My_Programming\...`, `G:\_My_Programming-2\...` — другие пути
- `G:\___Planning_Life_Sphere\...`, `G:\Projects_Life_planning\...`
- `session-*.json`
- git-remote URL
- VPS-пути `hermes@65.108.152.24:~/...` (Linux, не Windows)

**Ложные совпадения:** `"...string:\n"` -> `g:\` — не трогать.

### `.venv`

Не пересоздавать, не править — при запуске проекта.

---

## Шаг 4. Контрольный скан

После замены — в **операционных** файлах (`bat`, `ps1`, `py`, `yaml`, `env`) на F:
не должно остаться `G:\AI\_MY_PROGRAMMING`:

```
$ext = '\.(bat|ps1|py|yaml|yml|env)$'
$hits = @()
foreach ($root in @('F:\_MY_PROGRAMMING','F:\_MY_PROGRAMMING_4','F:\_MY_PROGRAMMING_IVRIT')) {
  Get-ChildItem $root -Recurse -File | Where-Object { $_.Extension -match $ext } |
    ForEach-Object {
      if (Select-String -Path $_.FullName -Pattern 'G:\\AI\\_MY_PROGRAMMING' -SimpleMatch -Quiet) {
        $hits += $_.FullName
      }
    }
}
"operational hits=$($hits.Count)"
$hits | ForEach-Object { $_ }
```

Ожидание: **0 hits**. Если есть — исправить вручную, пересканировать.

---

## Шаг 5. REPORT.md

1. Robocopy exit code + время выполнения.
2. Сверка src/dst (match=True).
3. `Changed files: N` из скрипта (3a + 3b).
4. Контрольный скан (0 hits).
5. Итоговый вердикт.

---

## Критерии приёмки

- [ ] robocopy exit 0–7, match=True (101019).
- [ ] Замена путей выполнена **без сабагента**, число изменённых файлов в отчёте.
- [ ] Операционные файлы: 0 x `G:\AI\_MY_PROGRAMMING` на F: (включая _MY_PROGRAMMING_4, _MY_PROGRAMMING_IVRIT).
- [ ] REPORT.md с полным выводом команд.
- [ ] Оригинал на G: **не удалён**.

## Чего НЕ делать

- Не удалять на G:.
- Не использовать сабагентов Hermes для замены путей.
- Не использовать `Format-Table` / `Format-List`.
- Не трогать другие папки на G:.
- Не переносить `_MY_PROGRAMMING_3` — отдельное задание позже.
