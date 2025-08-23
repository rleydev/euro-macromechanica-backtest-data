# Euro Macromechanica (EMM) Backtest — Data Hub (⚠️ third‑party datasets inside)

> 🧭 Этот репозиторий является частью экосистемы **Euro Macromechanica Backtest (EMM)**.

This repository stores raw market datasets (e.g., HistData), prepared files, compiled calendars,
and analysis aggregates used for **transparency and reproducibility** of the EMM backtest.

> **Legal:** rights in third‑party datasets remain with their providers. This repo does **not** re‑license them.
> See `LICENSES.md`, `NOTICE-THIRD-PARTY.*`, and `SOURCES.md`.

## Layout
```
source_data/        # raw third‑party datasets (original terms apply)
prepared/           # normalized/cleaned files derived from source_data
economic_calendars/          # compiled calendar metadata (UTC), if sources allow
analysis/           # aggregates: gaps, continuity metrics, summaries
integrity/          # artifacts.sha256 (+ .asc, .ots) per snapshot
SOURCES.md          # provenance (URL, date fetched, SHA‑256)
LICENSES.md         # license map by folders
NOTICE-THIRD-PARTY.md / .ru.md
DATA_USAGE.md / .ru.md
INTEGRITY.md
.gitattributes      
```
