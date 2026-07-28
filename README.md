# Brazil Mid-Market Deals Radar

A small, reproducible tracker of **Brazilian mid-market deal flow** — announced
M&A, private equity, venture, and select debt — read through a **credit lens**,
with a public sector-level credit-stress overlay.

The value on show is the **framework**: a clean data model, acquirer-name
canonicalization, validation, and a reproducible analysis that crosses deal
activity with a public credit-stress signal — not statistical breadth. I approach
this as a **deals generalist who brings a sharper credit lens to diligence**,
treating leverage and credit quality as part of how you read a transaction rather
than as a distressed-only specialty.

> **Portfolio / practice build.** The deal records here are **synthetic**,
> generated with a fixed seed. Every row carries `source = SYNTHETIC-EXAMPLE` and
> uses fictional placeholder names, so no real transaction is implied. To use this
> for real, replace the synthetic rows and log actual deals from **public** sources
> via the `add_deal(...)` helper in the notebook. The credit side, by contrast, is
> **real public data** from the Brazilian Central Bank.

## What's in here

| File | What it is |
|---|---|
| `deals_radar.ipynb` | **Phase 1** — generate/validate the deal log, canonicalize acquirers, and profile the most active acquirers over a trailing-12-month window. |
| `deals_radar_fase_2.ipynb` | **Phase 2** — build a per-sector credit-stress index from BCB SCR.data and cross it with deal activity. |
| `deals_radar_fase_3.ipynb` | **Phase 3** — bring a *second, independent* public credit lens (CVM corporate leverage) and cross it with the SCR stress by sector. Real on both axes. |
| `app.py` | Interactive **dashboard** (Streamlit) that presents Phases 1–2 with live filters. A delivery layer — reads the CSVs, fetches nothing. |
| `deals.csv` | The deal log (synthetic demo rows). |
| `stress_indicators.csv` | Per-sector credit stress, rebuilt from the BCB by the Phase 2 notebook. *Commit it so the app and Phase 3 run offline.* |
| `leverage_indicators.csv` | Per-sector corporate leverage, rebuilt from the CVM by the Phase 3 notebook. *Commit it so Phase 3 runs offline.* |
| `companies_leverage.csv` | Per-company leverage detail (for the Phase 3 drill-down), rebuilt from the CVM. *Commit it so the drill-down runs offline.* |
| `requirements.txt` | pandas, matplotlib, jupyter, requests, scipy (Phase 3), streamlit + ipywidgets (dashboards). |

## Run it

```bash
pip install -r requirements.txt
```

**Notebooks.** Open either notebook and run all cells top-to-bottom. Phase 2 loads
the committed `stress_indicators.csv` by default; set `REFRESH_STRESS = True` to
rebuild it from the BCB (downloads ~250 MB of annual SCR.data archives into
`.scr_cache/`, which is git-ignored).

**Dashboard.**

```bash
streamlit run app.py
```

The app reads `deals.csv` and `stress_indicators.csv` from the repo root and
renders two views — the deal-flow panorama and the credit × deals overlay — with
filters for window, acquirer type, and sector. It is a presentation layer only: it
never collects, scrapes, or recomputes anything. If `stress_indicators.csv` is
absent, the overlay tab explains how to produce it (run the Phase 2 notebook) and
the rest of the app still works.

## Phase 1 — Deal-flow panorama

A seeded generator fabricates clearly-labeled sample deals so the charts have
something to show — this is demo data, not collection. The notebook validates the
schema (types, ranges, unique/sequential ids), normalizes sectors through an
editable `SECTOR_MAP`, folds acquirer spelling variants onto `acquirer_canonical`,
restricts to the trailing 12 months anchored on the **latest** `announce_date`, and
draws two headline charts: most active acquirers **by deal count** and **by
aggregate disclosed value** (undisclosed values excluded, and the excluded count
reported).

## Phase 2 — Credit × deals overlay

A per-sector **credit stress index** from one public source: **BCB SCR.data**, which
publishes the corporate loan book and non-performing loans by CNAE section. Two
signals are drawn from it — the current NPL **level** and its **12-month trend** —
combined via z-score with the trend weighted heavier (0.6 vs 0.4). A deteriorating
book is more informative than a high but stable one. The index is joined to deal
activity and plotted as **activity × stress with quadrants**, surfacing the thesis
corner: active sectors where credit is turning.

The notebook rebuilds `stress_indicators.csv` straight from the BCB (set
`REFRESH_STRESS = True`), stamping each run with the SHA-256 of the source archives.
The derived CSV is committed, so a fresh clone runs offline with no download.

**A note on the NPL definition.** BCB defines non-performance as the *full balance*
of loans with any instalment more than 90 days overdue — not the overdue instalments
alone. The obvious-looking field (`vencido_acima_de_90_dias`) is the latter and
understates NPL by roughly half; the correct one is `carteira_inadimplencia`. The
index cross-checks itself against the BCB headline: it refuses to write a file if
aggregate corporate NPL lands outside a plausible band.

**Why SCR.data and not SGS.** SGS — the BCB source most people reach for — has no
sectoral corporate NPL series. Its corporate NPL series break down by loan
*modality* (vehicles, cards, working capital), not by economic activity. The
sectoral cut exists only in SCR.data.

**Reference reading (from real BCB data).** Aggregate corporate NPL sits around
2.8%, up from ~2.3% a year earlier; the CNAE→sector mapping covers ~86% of the
corporate loan book. Credit stress (Y) is real; deal activity (X) is synthetic and
labeled — the chart demonstrates the *method* of crossing the two, so the position
of any individual bubble is not a market call.

## Phase 3 — A second public source: SCR × CVM

Phase 3 brings an *independent* public credit lens and crosses it with the SCR
stress from Phase 2. The second source is the **CVM's open financial statements
(DFP)**: for B3-listed companies it computes sector-level **corporate leverage** —
net debt / equity, gross debt / assets, and net debt / EBITDA — each book-weighted
across the sector's companies, with the company's sector taken from the CVM
cadastral file (`SETOR_ATIV`) and mapped onto the radar's sectors.

The analytical question is whether the two lenses agree: where a sector's
**whole-universe loan book** (SCR NPL) and its **listed-name balance sheets** (CVM
leverage) both flag stress, the signal is corroborated; where they diverge, the
disagreement is itself informative (e.g., stress concentrated in smaller unlisted
firms the CVM sample never sees). Unlike the Phase 2 chart, **this cross is real on
both axes** — SCR and CVM are both public. The notebook opens the data five ways: a
rank-correlation **cross** and corroboration map, a **composite credit-stress score**
(both lenses + trends, z-weighted into one origination-priority ranking), a
**divergence** ranking that isolates where the lenses disagree, a **company
drill-down** naming the listed firms behind each sector's leverage, and a **leverage
trend** (two DFP vintages) crossed with the NPL trend to flag sectors deteriorating on
both signals at once. The notebook reuses the Phase 2 machinery: cache-first download, SHA-256
stamping, sanity gates that refuse to write an implausible result, and a committed
`leverage_indicators.csv` so a fresh clone runs offline (`REFRESH_LEVERAGE = True`
to rebuild from the CVM).

The main limitation is a **universe mismatch**: SCR spans every firm with registered
credit, CVM only listed large-caps, and the deals are mid-market — three
populations, so the cross is triangulation, not identity. EBITDA leans on an
approximated D&A, so `nd_ebitda` is indicative; the balance-sheet ratios are more
robust.

**Dashboard (delivery layer).** `app.py` turns the deal-flow and credit analyses
into a browsable Streamlit app with live filters, deployable to Streamlit Community
Cloud for a shareable link; `deals_radar_fase_3` can also be run as an interactive
`ipywidgets` view. These present the analysis — the notebooks remain the single
source of truth for the numbers.

## Data model

| Field | Notes |
|---|---|
| `deal_id` | Sequential integer |
| `announce_date` | ISO `YYYY-MM-DD` |
| `target` | Company acquired / invested in |
| `acquirer_raw` | Acquirer name as written in the source |
| `acquirer_canonical` | Normalized acquirer name — **the grouping key for all analysis** |
| `acquirer_type` | `PE` / `Estrategico` / `VC` / `FO` |
| `deal_type` | `M&A` / `PE` / `VC` / `divida` / `distressed` |
| `sector` | Target sector (normalized via `SECTOR_MAP`) |
| `value_brl_mm` | Disclosed value in R$ millions; blank if undisclosed |
| `stake_pct` | Stake acquired, %; blank if n/a |
| `source` | Publication (e.g., NeoFeed, Brazil Journal) |
| `source_url` | Link to the public source |
| `notes` | Author notes (credit / thesis observations) |

## Limitations

- **The deals are synthetic.** Every deal record is generated and labeled. No public
  feed of Brazilian mid-market transactions exists to draw on, so the deal side
  demonstrates the framework, not the market. The credit side is real.
- **Hybrid axes.** The Phase 2 chart crosses real credit stress (Y) with synthetic
  deal activity (X). It shows the method of crossing them, not a market call.
- **Taxonomy mapping.** CNAE sections do not map 1:1 onto deal sectors. The mapping
  is an explicit approximation; part of the corporate loan book is excluded, and Food
  & Beverage borrows the stress of its parent section, manufacturing (flagged with an
  asterisk on the chart).
- **Universe mismatch.** SCR covers every company with registered credit, from micro
  to large; these deals are mid-market, so a sector's NPL is pulled by its larger
  firms.
- **Publication lag.** SCR.data is released with a delay — a recent-past snapshot,
  not today.
- **Small reference set.** The z-score is computed across the ~11 mapped sectors, not
  the whole economy. Read the ranking, not the decimals.

## Sources

- BCB SCR.data — https://dadosabertos.bcb.gov.br/dataset/scr_data
- SCR methodology (v2) — https://www.bcb.gov.br/pda/desig/metodologia_versao2.pdf
- CVM Dados Abertos, DFP (financial statements) — https://dados.cvm.gov.br/dataset/cia_aberta-doc-dfp

## Data integrity

Public information only. No proprietary, employer, or confidential data of any kind.
Synthetic records are always clearly labeled; no real deals are fabricated; there are
no scrapers or automated collectors — deals are logged by hand from public sources.
