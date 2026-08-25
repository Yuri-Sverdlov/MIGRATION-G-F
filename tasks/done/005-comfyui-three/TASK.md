# TASK-05: перенос 3 ComfyUI-папок (SHARED + 2 гиганта)

> Задание: архитектор -> кодер. Отчёт пиши в `tasks/REPORT.md`.
> Правила среды: `AGENTS.md` (читай первым).

## Контекст

Партии 1, 2a, 2b, 4 закрыты. Переносим **три** папки ComfyUI-стека:

| # | SRC (G:) | DST (F:) | Файлов (эталон) | Режим |
|---|----------|----------|-----------------|-------|
| A | `G:\AI\COMFYUI-SHARED` | `F:\COMFYUI-SHARED` | **221** | обычный robocopy |
| B | `G:\AI\COMFYUI_PORTABLE` | `F:\COMFYUI_PORTABLE` | **57 472** | фон + `/MT:16` |
| C | `G:\AI\COMFY_UI` | `F:\COMFY_UI` | **43 294** | фон + `/MT:16` |

**Порядок обязателен:** A -> B -> C (PORTABLE ссылается на SHARED в `extra_model_paths.yaml`).

**Оригиналы на G: не удалять** — пользователь удалит после приёмки.

**Важно:** сабагенты Hermes **не использовать**. Замену путей делать **самому** — PowerShell-скриптом.

**Уже на F: (не копировать, только знать):**
- `F:\COMFYUI-SYNCHRONIZATION\install_path.txt` уже содержит `F:\COMFYUI_PORTABLE`.
- `F:\_MY_PROGRAMMING_2\COMFYUI-WAN2-2\*.bat` уже ссылаются на `F:\COMFYUI_PORTABLE`.

---

## Фаза A. COMFYUI-SHARED (221 файлов)

### A1. Robocopy

```
robocopy "G:\AI\COMFYUI-SHARED" "F:\COMFYUI-SHARED" /E /COPY:DAT /R:1 /W:1
```

Exit code 0–7 = успех; 8+ = стоп + REPORT.

### A2. Проверка

```
$src = (Get-ChildItem "G:\AI\COMFYUI-SHARED" -Recurse -File).Count
$dst = (Get-ChildItem "F:\COMFYUI-SHARED" -Recurse -File).Count
"COMFYUI-SHARED -> src=$src dst=$dst match=$($src -eq $dst)"
```

Ожидание: **221 = 221**, match=True.

### A3. Замена путей в `F:\COMFYUI-SHARED`

Ключевой файл: `tools\comfy_shared\config.py` — пути `G:\AI\...` -> `F:\...`.
Также: `extra_model_paths.yaml` (`G:/AI/COMFYUI-SHARED` -> `F:/COMFYUI-SHARED`).

**WORKING_COMFYUI:** было `G:\AI\_MY_PROGRAMMING_2\ComfyUI` (призрак) -> **`F:\_MY_PROGRAMMING_2\ComfyUI`**
(папка `_MY_PROGRAMMING_2` уже на F:).

**SCAN_ROOTS:** заменить только элементы под `G:\AI\`:
- `G:\AI\_MY_PROGRAMMING_2` -> `F:\_MY_PROGRAMMING_2`
- `G:\AI\COMFYUI_PORTABLE` -> `F:\COMFYUI_PORTABLE`
- `G:\AI\MODEL-SORT` -> `F:\MODEL-SORT`
- `G:\AI\COMFYUI-SHARED` -> `F:\COMFYUI-SHARED`

**НЕ менять:** `G:\Video\...`, `G:\DiskU\...`, `S:\...` — другие диски.

---

## Фаза B. COMFYUI_PORTABLE (57 472 файла)

### B1. Robocopy (фон, /MT:16)

```
robocopy "G:\AI\COMFYUI_PORTABLE" "F:\COMFYUI_PORTABLE" /E /COPY:DAT /R:1 /W:1 /MT:16
```

Запускать **в фоне**. Ждать завершения перед B2.

### B2. Проверка

```
$src = (Get-ChildItem "G:\AI\COMFYUI_PORTABLE" -Recurse -File).Count
$dst = (Get-ChildItem "F:\COMFYUI_PORTABLE" -Recurse -File).Count
"COMFYUI_PORTABLE -> src=$src dst=$dst match=$($src -eq $dst)"
```

Ожидание: **57 472 = 57 472**, match=True.

### B3. Замена путей в `F:\COMFYUI_PORTABLE`

Операционные файлы (не `site-packages`, не `.git`):
- `ComfyUI\extra_model_paths.yaml`: `G:/AI/COMFYUI-SHARED` -> `F:/COMFYUI-SHARED`
- `.claude\settings.local.json`: `G:/AI/COMFYUI_PORTABLE` -> `F:/COMFYUI_PORTABLE`,
  `G:/AI/COMFYUI-SYNCHRONIZATION` -> `F:/COMFYUI-SYNCHRONIZATION`

Launcher-скрипты (`run_nvidia_gpu.bat` и др.) — относительные пути, **не трогать**.

---

## Фаза C. COMFY_UI (43 294 файла)

### C1. Robocopy (фон, /MT:16)

```
robocopy "G:\AI\COMFY_UI" "F:\COMFY_UI" /E /COPY:DAT /R:1 /W:1 /MT:16
```

Запускать **в фоне**. Ждать завершения перед C2.

### C2. Проверка

```
$src = (Get-ChildItem "G:\AI\COMFY_UI" -Recurse -File).Count
$dst = (Get-ChildItem "F:\COMFY_UI" -Recurse -File).Count
"COMFY_UI -> src=$src dst=$dst match=$($src -eq $dst)"
```

Ожидание: **43 294 = 43 294**, match=True.

### C3. Замена путей в `F:\COMFY_UI`

В `extra_models_config.yaml` (2 копии: корень и `ComfyUI_windows_portable_nvidia\`) —
**устаревший путь** `G:\С 2ТБ\COMFY_UI\...` (кириллица в имени диска).

**Не передавать кириллический путь строкой в команде.** Искать файлы по ASCII-имени:

```
Get-ChildItem 'F:\COMFY_UI' -Recurse -File -Filter 'extra_models_config.yaml'
```

Замена через regex (любой `G:\...\COMFY_UI` -> `F:\COMFY_UI`):

```powershell
Get-ChildItem 'F:\COMFY_UI' -Recurse -File -Filter 'extra_models_config.yaml' |
  ForEach-Object {
    $path = $_.FullName
    $text = [IO.File]::ReadAllText($path)
    $new = $text -replace 'G:\\[^\\]+\\COMFY_UI', 'F:\COMFY_UI'
    if ($new -ne $text) {
      [IO.File]::WriteAllText($path, $new, [Text.UTF8Encoding]::new($false))
      "Updated: $path"
    }
  }
```

Проверить также `comfy.settings.json` и `.bat` в корне portable — если есть `G:\AI\` или старый `G:\...\COMFY_UI`, исправить.

**НЕ сканировать/менять** `python_embeded\Lib\site-packages\` — сторонние библиотеки.

---

## Шаг 4. Общий скрипт замены G:\AI\ (фазы A + B)

После копирования SHARED и PORTABLE — прогнать на обеих папках:

```powershell
$roots = @('F:\COMFYUI-SHARED', 'F:\COMFYUI_PORTABLE')
$extPattern = '\.(py|ps1|bat|json|env|md|txt|yml|yaml|cfg|ini)$'
$skipParts = @('\site-packages\', '\.git\', '\python_embeded\Lib\')

$replacements = @(
  @{ Old = 'G:/AI/COMFYUI-SHARED/';     New = 'F:/COMFYUI-SHARED/' },
  @{ Old = 'G:/AI/COMFYUI_PORTABLE/';   New = 'F:/COMFYUI_PORTABLE/' },
  @{ Old = 'G:/AI/COMFYUI-SYNCHRONIZATION/'; New = 'F:/COMFYUI-SYNCHRONIZATION/' },
  @{ Old = 'G:/AI/_MY_PROGRAMMING_2/';  New = 'F:/_MY_PROGRAMMING_2/' },
  @{ Old = 'G:/AI/MODEL-SORT/';         New = 'F:/MODEL-SORT/' },
  @{ Old = 'G:\\AI\\COMFYUI-SHARED\\';  New = 'F:\\COMFYUI-SHARED\\' },
  @{ Old = 'G:\\AI\\COMFYUI_PORTABLE\\'; New = 'F:\\COMFYUI_PORTABLE\\' },
  @{ Old = 'G:\\AI\\COMFYUI-SYNCHRONIZATION\\'; New = 'F:\\COMFYUI-SYNCHRONIZATION\\' },
  @{ Old = 'G:\\AI\\_MY_PROGRAMMING_2\\'; New = 'F:\\_MY_PROGRAMMING_2\\' },
  @{ Old = 'G:\\AI\\MODEL-SORT\\';      New = 'F:\\MODEL-SORT\\' },
  @{ Old = 'G:\AI\COMFYUI-SHARED\';     New = 'F:\COMFYUI-SHARED\' },
  @{ Old = 'G:\AI\COMFYUI_PORTABLE\';   New = 'F:\COMFYUI_PORTABLE\' },
  @{ Old = 'G:\AI\COMFYUI-SYNCHRONIZATION\'; New = 'F:\COMFYUI-SYNCHRONIZATION\' },
  @{ Old = 'G:\AI\_MY_PROGRAMMING_2\';  New = 'F:\_MY_PROGRAMMING_2\' },
  @{ Old = 'G:\AI\MODEL-SORT\';         New = 'F:\MODEL-SORT\' },
  @{ Old = 'G:\AI\COMFYUI-SHARED';      New = 'F:\COMFYUI-SHARED' },
  @{ Old = 'G:\AI\COMFYUI_PORTABLE';    New = 'F:\COMFYUI_PORTABLE' },
  @{ Old = 'G:\AI\COMFYUI-SYNCHRONIZATION'; New = 'F:\COMFYUI-SYNCHRONIZATION' },
  @{ Old = 'G:\AI\_MY_PROGRAMMING_2';   New = 'F:\_MY_PROGRAMMING_2' },
  @{ Old = 'G:\AI\MODEL-SORT';          New = 'F:\MODEL-SORT' },
  @{ Old = 'g:/AI/COMFYUI-SHARED/';     New = 'f:/COMFYUI-SHARED/' },
  @{ Old = 'g:/AI/COMFYUI_PORTABLE/';   New = 'f:/COMFYUI_PORTABLE/' },
  @{ Old = 'g:\AI\COMFYUI-SHARED\';     New = 'f:\COMFYUI-SHARED\' },
  @{ Old = 'g:\AI\COMFYUI_PORTABLE\';   New = 'f:\COMFYUI_PORTABLE\' }
)

$totalChanged = 0
foreach ($root in $roots) {
  $changed = @()
  Get-ChildItem $root -Recurse -File -ErrorAction SilentlyContinue |
    Where-Object {
      $_.Extension -match $extPattern -and
      ($skipParts | Where-Object { $_.FullName -like "*$_*" }).Count -eq 0
    } |
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
  Write-Output "$root Changed files: $($changed.Count)"
  $totalChanged += $changed.Count
}
Write-Output "Total changed: $totalChanged"
```

Комментарии в `.md` (_planning) с упоминанием `G:\AI\...` — **можно** оставить (история), но `config.py` и `.yaml` — **обязательно** исправить.

---

## Шаг 5. Контрольный скан

В **операционных** файлах (`bat`, `ps1`, `py`, `yaml`, `yml`, `json`, `env`) на F:
не должно остаться `G:\AI\COMFYUI-SHARED`, `G:\AI\COMFYUI_PORTABLE`, `G:/AI/COMFYUI`:

```
$patterns = @('G:\AI\COMFYUI-SHARED','G:\AI\COMFYUI_PORTABLE','G:/AI/COMFYUI-SHARED','G:/AI/COMFYUI_PORTABLE')
$ext = '\.(bat|ps1|py|yaml|yml|json|env)$'
$skipParts = @('\site-packages\', '\.git\', '\python_embeded\Lib\', '\_planning\')
$hits = @()
foreach ($root in @('F:\COMFYUI-SHARED','F:\COMFYUI_PORTABLE','F:\COMFY_UI')) {
  Get-ChildItem $root -Recurse -File -ErrorAction SilentlyContinue |
    Where-Object {
      $_.Extension -match $ext -and
      ($skipParts | Where-Object { $_.FullName -like "*$_*" }).Count -eq 0
    } |
    ForEach-Object {
      foreach ($p in $patterns) {
        if (Select-String -Path $_.FullName -Pattern $p -SimpleMatch -Quiet) {
          $hits += $_.FullName
          break
        }
      }
    }
}
"operational hits=$($hits.Count)"
$hits | ForEach-Object { $_ }
```

Дополнительно — legacy COMFY_UI на F::

```
$legacy = @()
Get-ChildItem 'F:\COMFY_UI' -Recurse -File -Filter 'extra_models_config.yaml' |
  ForEach-Object {
    if (Select-String -Path $_.FullName -Pattern '\\COMFY_UI' -Quiet) {
      $t = Get-Content $_.FullName -Raw
      if ($t -notmatch 'F:\\COMFY_UI') { $legacy += $_.FullName }
    }
  }
"legacy COMFY_UI path hits=$($legacy.Count)"
```

Ожидание: **0 hits** по обоим сканам.

---

## Шаг 6. REPORT.md

Для каждой фазы (A, B, C):
1. Robocopy exit code + время.
2. Сверка src/dst (match=True, эталонные числа).
3. Число изменённых файлов (скрипт + ручные правки).
4. Контрольный скан (0 hits).
5. Итоговый вердикт.

---

## Критерии приёмки

- [ ] robocopy exit 0–7 для всех трёх; match=True (221, 57472, 43294).
- [ ] `config.py`: SHARED_ROOT и SCAN_ROOTS указывают на F:\.
- [ ] `extra_model_paths.yaml` (SHARED + PORTABLE): `base_path` -> `F:/COMFYUI-SHARED/models/`.
- [ ] `extra_models_config.yaml` (COMFY_UI): legacy `G:\...\COMFY_UI` -> `F:\COMFY_UI`.
- [ ] Замена путей **без сабагента**; число файлов в отчёте.
- [ ] Операционные файлы: 0 x `G:\AI\COMFYUI-*` на F:.
- [ ] REPORT.md с полным выводом команд.
- [ ] Оригиналы на G: **не удалены**.

## Чего НЕ делать

- Не удалять на G:.
- Не использовать сабагентов Hermes.
- Не использовать `Format-Table` / `Format-List`.
- Не трогать `_MY_PROGRAMMING_3`, другие папки вне списка.
- Не править `site-packages` и `.venv`.
- Не менять пути на других дисках (`G:\Video`, `G:\DiskU`, `S:\`).
