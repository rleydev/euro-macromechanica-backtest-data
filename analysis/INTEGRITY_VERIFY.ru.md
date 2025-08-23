# Integrity Verification — Hashes, GPG, OpenTimestamps (with absolute paths support)

Этот файл описывает **как верифицировать годовой снапшот** (входы/выходы бэктеста) по манифесту хэшей,
подписи GPG и якорю OpenTimestamps, **если в `artifacts_YYYY.sha256` указаны абсолютные пути**.

## 📦 Состав снапшота (по умолчанию)
- **Манифесты целостности:**  
  `integrity/artifacts_YYYY.sha256`, `integrity/artifacts_YYYY.sha256.asc` (GPG), `integrity/artifacts_YYYY.sha256.ots` (OpenTimestamps).
- **Отчёт:** `analysis/output_bundle_report/annual_report_YYYY.md`  
  В нём *напечатан* `— Manifest SHA-256: <…>` — это **SHA-256 самого манифеста**.
- **Входы:** `source_data/**`, `economic_calendars/**`  
- **Выходы:** `prepared/**`, `analysis/output_bundle_report/**`

> Папки могут отличаться от проекта к проекту — ориентируйтесь на свой layout и манифест.

---

## TL;DR
1. **Проверьте подпись GPG и (опционально) OTS** для `artifacts_YYYY.sha256`.
2. **Сверьте, что в отчёте напечатан SHA-256 манифеста.**
3. **Сравните SHA нормализованного отчёта** (строка `— Manifest SHA-256:` *без* значения) с записью в манифесте.
4. Для удобной сверки всех файлов **создайте локальную копию манифеста с basename** (absolute → basename) и проверьте по ней.

> Абсолютные пути **не влияют** на правильность хэшей; они лишь мешают командам, которые ждут относительные пути.

---

## 🧪 Сверка хэшей входных/выходных файлов с манифестом

Входные файлы — в **`/source_data`** и **`/economic_calendars`**, выходные — в **`/prepared`** и **`/analysis/output_bundle_report`**.

### 🐧 macOS / Linux (bash/zsh)

> Если нет `sha256sum`, используйте `shasum -a 256`. Переменная `HASH` выберет нужное.

```bash
# 0) выбрать хэш-команду
if command -v sha256sum >/dev/null 2>&1; then HASH='sha256sum'; else HASH='shasum -a 256'; fi
echo "Using: $HASH"
```

**0.a) Нормализовать пути манифеста (absolute → basename)**  
Создаём локальную копию манифеста, где вместо абсолютных путей — только имена файлов:
```bash
YEAR=YYYY
sed -E 's#^([0-9a-f]{64})  (.+[/\\])?([^/\\]+)$#\1  \3#' artifacts_${YEAR}.sha256 > artifacts_${YEAR}.local.sha256
```
> Это не меняет сами хэши — только «ярлык пути». Убедитесь, что **имена файлов уникальны**. Если есть дубликаты имён в разных папках, их нужно предварительно переименовать или сверять по отдельным локальным манифестам.

**1) Хэш манифеста должен быть напечатан в отчёте (внешняя метка)**
```bash
MAN_SHA=$($HASH artifacts_${YEAR}.sha256 | awk '{print $1}')
grep -F "— Manifest SHA-256: $MAN_SHA" annual_report_${YEAR}.md >/dev/null \
  && echo "OK: отчёт содержит хэш манифеста" \
  || echo "BAD: нет хэша манифеста в отчёте"
```

**2) Сравнить нормализованный SHA отчёта с записью в манифесте**  
Извлекаем из **оригинального** манифеста строку для отчёта, даже если путь абсолютный:
```bash
MAN_REP_SHA=$(grep -E '^([0-9a-f]{64})  (.+[/\\])?annual_report_'${YEAR}'\.md$' artifacts_${YEAR}.sha256 | head -n1 | awk '{print $1}')

# Нормализуем отчёт (убираем значение после "Manifest SHA-256:") и считаем его SHA-256
REP_NORM_SHA=$(sed "s/— Manifest SHA-256: $MAN_SHA/— Manifest SHA-256:/" annual_report_${YEAR}.md \
  | tr -d '\r' \
  | $HASH | awk '{print $1}')

[ "$MAN_REP_SHA" = "$REP_NORM_SHA" ] \
  && echo "OK: нормализованный SHA отчёта совпал с манифестом" \
  || echo "BAD: отчёт не совпал"
```

**3) Сверить все остальные файлы из манифеста (без отчёта)**  
Работаем по **локальной** версии манифеста (с basename), чтобы `-c` находил файлы:
```bash
# пересчитать локально
awk '$2!="annual_report_'${YEAR}'.md"{print $2}' artifacts_${YEAR}.local.sha256 \
  | xargs -I{} $HASH "{}" \
  | awk '{print $1"  "$2}' | sort -k2 > /tmp/recalc.txt

# привести манифест к тому же виду и отсортировать
awk '$2!="annual_report_'${YEAR}'.md"{print}' artifacts_${YEAR}.local.sha256 \
  | sort -k2 > /tmp/manifest_no_report.txt

diff -u /tmp/manifest_no_report.txt /tmp/recalc.txt \
  && echo "OK: все файлы совпали" \
  || echo "BAD: есть расхождения"
```

> Альтернатива: одним махом по локальному манифесту:  
> ```bash
> $HASH -c artifacts_${YEAR}.local.sha256
> ```
> (при необходимости уберите строку отчёта из `.local.sha256`).

---

### 🪟 Windows PowerShell

**0) Сделать локальную копию манифеста с basename**
```powershell
$YEAR = "YYYY"
Get-Content "artifacts_$YEAR.sha256" | % {
  if ($_ -match '^([0-9a-f]{64})\s\s(.+)$') {
    "$($matches[1])  $(Split-Path $matches[2] -Leaf)"
  }
} | Set-Content "artifacts_$YEAR.local.sha256"
```

**1) Хэш манифеста должен быть напечатан в отчёте**
```powershell
$manHash = (Get-FileHash ".\artifacts_$YEAR.sha256" -Algorithm SHA256).Hash.ToLower()
if ((Get-Content ".\annual_report_$YEAR.md" -Raw) -match "— Manifest SHA-256:\s*$manHash") {
  "OK: отчёт содержит хэш манифеста"
} else {
  "BAD: нет хэша манифеста в отчёте"
}
```

**2) Сравнить нормализованный SHA отчёта с записью в манифесте (путь мог быть абсолютным)**
```powershell
# извлечь SHA для annual_report_* из ОРИГИНАЛЬНОГО манифеста (путь может быть любой)
$line = Select-String -Path ".\artifacts_$YEAR.sha256" -Pattern '^([0-9a-f]{64})\s\s(.+\\)?annual_report_' + $YEAR + '\.md$' | Select-Object -First 1
$manRepHash = $line.Matches.Groups[1].Value.ToLower()

# нормализуем отчёт и считаем SHA-256
$report = Get-Content ".\annual_report_$YEAR.md" -Raw
$norm   = [regex]::Replace($report, "— Manifest SHA-256:\s*[0-9A-Fa-f]{64}", "— Manifest SHA-256:")
$sha256 = [System.Security.Cryptography.SHA256]::Create()
$bytes  = [System.Text.Encoding]::UTF8.GetBytes($norm)
$repHash = ($sha256.ComputeHash($bytes) | ForEach-Object ToString x2) -join "" | ForEach-Object ToLower

if ($repHash -eq $manRepHash) { "OK: нормализованный SHA отчёта совпал" } else { "BAD: отчёт не совпал" }
```

**3) Сверить все остальные файлы из локального манифеста (без отчёта)**
```powershell
$errors = @()
Get-Content ".\artifacts_$YEAR.local.sha256" | ForEach-Object {
  if ($_ -match '^\s*$') { return }
  $parts = $_ -split '\s{2}',2
  $hashRef = $parts[0].ToLower()
  $name    = $parts[1]
  if ($name -eq "annual_report_$YEAR.md") { return }
  if (-not (Test-Path $name)) { $errors += "MISSING: $name"; return }
  $hashAct = (Get-FileHash $name -Algorithm SHA256).Hash.ToLower()
  if ($hashAct -ne $hashRef) { $errors += "MISMATCH: $name`n  manifest=$hashRef`n  actual  =$hashAct" }
}
if ($errors.Count -eq 0) { "OK: все файлы совпали" } else { $errors -join "`n" }
```

---

## 🔐 Подпись GPG и OpenTimestamps

```bash
# проверка GPG-подписи манифеста
gpg --import keys/emm_pub_key.asc
gpg --verify integrity/artifacts_YYYY.sha256.asc integrity/artifacts_YYYY.sha256

# проверка OpenTimestamps
ots verify integrity/artifacts_YYYY.sha256    # либо: ots verify integrity/artifacts_YYYY.sha256.ots
```

> Рекомендуется хранить рядом **оригинальный** манифест (с абсолютными путями) и **локальную копию** (`*.local.sha256`) для удобных проверок `-c`.

---

## 🧠 Частые вопросы

- **Абсолютные пути ломают проверку?**  
  Нет. Хэш считается **только от содержимого файла**. Пути — это подсказка для команды `-c`. Поэтому мы используем локальную копию с basename.

- **CRLF/LF?**  
  Для нормализации отчёта в примерах мы удаляем `\r` (`tr -d '\r'`), чтобы SHA совпадал независимо от платформы.

- **Дубликаты имён?**  
  Если два разных файла имеют один и тот же basename, создайте **отдельные локальные манифесты** для соответствующих подпапок или переименуйте файлы.

- **Что именно печатается в отчёте?**  
  В отчёте печатается **SHA-256 манифеста**. В манифесте, в свою очередь, хранится **SHA-256 нормализованного отчёта** (где строка `— Manifest SHA-256:` оставлена без значения).

- **Порядок строк важен?**  
  Для `-c` — нет. Для диффов в примере мы сортируем по имени файла.

---

