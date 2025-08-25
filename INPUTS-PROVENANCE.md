# 🔗 Inputs — Provenance & Binding — Binding input data to Euro Macromechanica (EMM) Backtest results

This document records **exactly how** the input data from the repository **euro-macromechanica-backtest-data** – *Data Hub* (raw / prepared / calendars)
are matched with their use in the public backtest results **euro-macromechanica-results** (*Results*). It describes the **binding logic** and **tables for OTS anchors** (SHA‑256, txid, block, UTC time).  
For detailed hash‑verification commands see `INTEGRITY.md` (root) – for raw HistData data and economic calendars. See `analysis/INTEGRITY_VERIFY.md` (the normalization nuance) – for checking the analysis manifest against reports, SVG, and the input files (raw HistData and calendars).

---

## 🎯 Purpose
- Specify **which inputs** (raw minute data, prepared minute data, calendars) were used.
- Show **where hashes/signatures/OTS** are stored that prove the **point in time** and **integrity** of the inputs.
- Provide a **simple mapping** between **euro-macromechanica-backtest-data** – *Data Hub* and **euro-macromechanica-results** – *Results*.

## 📦 Scope (what counts as “inputs”)
- **RAW minute data** (*HistData*): `source_data/**`
- **PREPARED minute data** (normalized/deduplicated, UTC): `source_data/prepared/**`
- **Economic calendars** (aggregated metadata, UTC): `economic_calendars/**`

> ⚠️ Rights to the **RAW** datasets remain with their original providers. This repository does **not** re‑license them.

---

## 🧩 Binding logic

### 1) RAW → PREPARED in euro-macromechanica-backtest-data (*Data Hub*)
- PREPARED files are generated from RAW by the analyzer and are accompanied by **analysis manifests** `analysis/YYYY/output_bundle_report/artifacts_YYYY.sha256` (mapping “source → result”). 
- The verification workflow is described in [`/analysis/INTEGRITY_VERIFY.md`](https://github.com/rleydev/euro-macromechanica-backtest-data/blob/main/analysis/INTEGRITY_VERIFY.md)
- Thus the PREPARED data are **cryptographically bound** to a specific RAW set.
> 📌 **The analyzer has two main stages: preparation of minute data and gap analysis. Economic calendars are also bound to the analysis manifest because they participate in the gap analysis. They can be verified against the analysis manifest, which confirms a complete linkage of all components at a point in time.**


### 2) euro-macromechanica-backtest-data – PREPARED → euro-macromechanica-results – Results (annual runs)
- In *Results* each annual run has its own `artifacts_YYYY.sha256` (plus `.asc`, `.ots` for HEADLINE), listing **all input files used** (including PREPARED minutes and economic calendars).  
- How to check the binding:
  1. Compare the SHA‑256 of PREPARED files listed in **euro-macromechanica-results** – `results/**/artifacts_YYYY.sha256`, with the SHA‑256 of the corresponding files in `source_data/prepared/YYYY**` in **euro-macromechanica-backtest-data** – (*Data Hub*).
  2. Do the same for calendar files: compare hashes from **euro-macromechanica-results** – `results/**/artifacts_YYYY.sha256` with files in `economic_calendars/YYYY/**` in **euro-macromechanica-backtest-data** – (*Data Hub*).

### 3) Archive roll‑ups (OTS time anchors)
- To simplify audit, we use **archives (tar.gz/tar.xz)** and **archive snapshots (SHA‑256, GPG and OTS)** (roll‑up) for key input sets: “all RAW”, “all PREPARED”, “all calendars”.  
- Their SHA‑256, GPG and OTS are published in **euro-macromechanica-backtest-data** under `integrity/**`. These archive snapshots **do not replace** per‑component manifests; instead they provide
  **proof of the dataset’s state at a point in time** (Bitcoin anchor).  
- An auditor:
  - verifies that files unpacked from the archives match the hashes in the current folders (`source_data/`, `prepared/`, `economic_calendars/`),  
  - and that the **yearly manifests in Results** reference exactly those files (see step 2).

> 🧠 In sum: the chain “RAW → PREPARED → Results” is confirmed by a combination of **per‑component manifests** and **archive snapshots with OTS**.

---

## 🧷 Anchor tables

### A) RAW minute data (archive snapshot)

|----------------------|--------------------------------------------|
| Archive              | **`<source_data_2001-2025(jul).tar.xz>`**  |
| Contents             | “Raw minute data (M1) from HistData; unmodified” |
| Hash function        | `SHA-256` |
| Archive SHA‑256      | **`<7654b325da63fb415ad0a1aef17daf1a9f1ea721d0bf50413eacdd3d5c5bab7d>`** |
| GPG signature (`.asc`) | `integrity/source_data/source_data_2001-2025(jul).tar.xz.sha256.asc` |
| OTS (`.ots`)         | `integrity/source_data/source_data_2001-2025(jul).tar.xz.sha256.ots` |
| Bitcoin txid         | **`<d883cb3a197d8ed47a184b4dcd305047cedae9a0ad948bd7993737af571ce088>`** |
| Block height         | **`<909599>`** |
| Block time (UTC)     | **`<2025-08-11 19:21:02>`** |
| Link (data‑hub)      | [`source_data/source_data_2001-2025(jul).tar.xz`](https://github.com/rleydev/euro-macromechanica-backtest-data/blob/main/source_data/source_data_2001-2025(jul).tar.xz) |

---

### B) PREPARED minute data (archive snapshot)

|----------------------|--------------------------------------------|
| Archive              | **`<analyzed_full_2001-2025jul.tar.gz>`**  |
| Contents             | “Prepared minute data (UTC±00:00) after filter/normalization” |
| Hash function        | `SHA-256` |
| Archive SHA‑256      | **`<e81f26072b48e62eb37dcd4da9919608e05da871b7264d3718e01244a2658d5f>`** |
| GPG signature (`.asc`) | `integrity/prepared/analyzed_full_2001-2025jul.tar.gz.sha256.asc` |
| OTS (`.ots`)         | `integrity/prepared/analyzed_full_2001-2025jul.tar.gz.sha256.ots` |
| Bitcoin txid         | **`<3646771ecc29d275b1b80ead9d5866a3b8867e6488df8d42145e2824a06e2a71>`** |
| Block height         | **`<910536>`** |
| Block time (UTC)     | **`<2025-08-18 02:06:26 >`** |
| Link (data‑hub)      | [`prepared/analyzed_full_2001-2025jul.tar.gz`](https://github.com/rleydev/euro-macromechanica-backtest-data/blob/main/prepared/analyzed_full_2001-2025jul.tar.gz) |

### C) Economic calendars (archive snapshot)

|----------------------|--------------------------------------------|
| Archive              | **`<jan2001-jul2025.tar.gz>`**  |
| Contents             | “Calendar metadata (central banks/statistical agencies), publication times in UTC” |
| Hash function        | `SHA-256` |
| Archive SHA‑256      | **`<0bd28458c49affc3b052116279827a8c840edf2dc4295d57a0b7e2a7b0b2e045>`** |
| GPG signature (`.asc`) | `integrity/economic_calendars/jan2001-jul2025.tar.gz.sha256.asc` |
| OTS (`.ots`)         | `integrity/economic_calendars/jan2001-jul2025.tar.gz.sha256.ots` |
| Bitcoin txid         | **`<a95e00f437575dad58c5e074352b6f4c9637ace705eccb05aa936e1cb99c6ffb>`** |
| Block height         | **`<910177>`** |
| Block time (UTC)     | **`<2025-08-15 16:31:10 >`** |
| Link (data‑hub)      | [`economic_calendars/jan2001-jul2025.tar.gz`](https://github.com/rleydev/euro-macromechanica-backtest-data/blob/main/economic_calendars/jan2001-jul2025.tar.gz) |

---

## 🔍 Where to find integrity artifacts
- **Root `INTEGRITY.md`** — commands to verify SHA‑256/GPG/OTS, including the note about **absolute paths** in manifests (how to “rebase” to relative paths for `sha256sum -c`).  
- **`analysis/INTEGRITY_VERIFY.md`** — the special note about **normalizing a single line** in the annual report before hashing (so it matches the manifest).

---

## 📝 Notes
- Manifests may contain **absolute paths** — this does not affect hash correctness. For local checks, rebase paths to relative (see `INTEGRITY.md`).  
- This file describes the **binding logic**. Detailed scripts/commands are in `INTEGRITY.md` and in the READMEs of the respective folders.
- Third‑party data remain under their providers’ terms; see `LICENSES.md` and `NOTICE-THIRD-PARTY*`.

---

## 🔗 Quick links
- Root verification guide: [`INTEGRITY.md`](https://github.com/rleydev/euro-macromechanica-backtest-data/tree/main/INTEGRITY.md)
- Analysis report verification nuance: [`analysis/INTEGRITY_VERIFY.md`](https://github.com/rleydev/euro-macromechanica-backtest-data/tree/main/analysis/INTEGRITY_VERIFY.md)
- RAW data: [`source_data/`](https://github.com/rleydev/euro-macromechanica-backtest-data/tree/main/source_data/)
- Prepared minutes: [`prepared/`](https://github.com/rleydev/euro-macromechanica-backtest-data/tree/main/prepared/)
- Calendars: [`economic_calendars/`](https://github.com/rleydev/euro-macromechanica-backtest-data/tree/main/economic_calendars/)
- Results (yearly manifests): see **euro-macromechanica-results** – *Results* (`artifacts_YYYY.sha256` + `.asc` + `.ots` for HEADLINE)
