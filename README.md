# Brazil Mid-Market Deals Radar

A small, reproducible tracker of **Brazilian mid-market deal flow** — announced
M&A, private equity, venture, and select debt — read through a credit lens.
**The whole project is one notebook: `deals_radar.ipynb`.** Open it and *Run All*.

It's a **framework** more than a dataset: a clean data model, validation,
acquirer-name canonicalization, and a reproducible analysis that profiles the
**most active acquirers** over a trailing 12-month window, plus a scaffold for a
later **public** credit-stress overlay. I approach this as a **deals generalist
who brings a sharper credit lens to diligence** — treating leverage and credit
quality as part of how you read a transaction, not as a distressed-only specialty.

> **Portfolio / practice build.** The dataset is **synthetic**, generated inside
> the notebook with a fixed seed. Every row carries `source = SYNTHETIC-EXAMPLE`
> with fictional placeholder names, so no real transaction is implied. Read it as
> **a reproducible deal-tracking framework demonstrated on synthetic data** — the
> value on show is the engineering. To use it for real, log actual deals from
> **public** sources via the `add_deal(...)` helper in the notebook.

## Run it

```bash
pip install -r requirements.txt      # pandas + matplotlib
jupyter notebook deals_radar.ipynb   # then Run All
```

The notebook runs top-to-bottom: it generates the synthetic dataset (only if
`deals.csv` doesn't already exist), validates it, cleans it, renders the two
headline charts inline, prints profile cuts, and exposes `add_deal(...)` for
logging real, publicly-sourced deals.

## What it produces

Most active acquirers by **deal count** and by **aggregate disclosed value**
(R$mm) over the trailing 12 months — undisclosed values excluded, with the
excluded count reported. Both are also written to `figures/`.

![Most active acquirers by deal count](figures/acquirers_by_count.png)

![Most active acquirers by disclosed value](figures/acquirers_by_value.png)

*(Generated from the synthetic demo data.)*

## Data model

One row per announced transaction:

| Field | Notes |
|---|---|
| `deal_id` | Sequential integer. |
| `announce_date` | ISO `YYYY-MM-DD`. |
| `target` | Company acquired / invested in. |
| `acquirer_raw` | Acquirer name as written in the source. |
| `acquirer_canonical` | Normalized acquirer name — **the grouping key for all analysis**. |
| `acquirer_type` | `PE` / `Estrategico` / `VC` / `FO`. |
| `deal_type` | `M&A` / `PE` / `VC` / `divida` / `distressed`. |
| `sector` | Target sector (normalized in-notebook via `SECTOR_MAP`). |
| `value_brl_mm` | Disclosed value, R$ millions; blank if undisclosed. |
| `stake_pct` | Stake acquired (0–100); blank if n/a. |
| `source` / `source_url` | Public publication and link. |
| `notes` | Author notes — where credit / thesis observations live. |

`acquirer_raw` preserves how the source wrote the name; `acquirer_canonical` is
what every chart groups on. `add_deal(...)` suggests a canonical name
(fuzzy-matched against existing ones) so near-duplicates don't creep in.

## Methodology

Trailing-12-month window anchored on the latest `announce_date`; grouping on
`acquirer_canonical` (with a printed spot-check of folded spellings); the value
chart sums only disclosed rows and reports exclusions; an editable `SECTOR_MAP`
tidies sectors with pass-through for anything unmapped. Manual entry only — no
scrapers or automated collectors anywhere.

## Roadmap

- **Phase 1 — Deal-flow panorama.** *(this notebook)*
- **Phase 2 — Credit × deals overlay.** Overlay **public** sector-stress signals
  (BCB NPLs, CVM/B3 leverage, Serasa recovery filings, IBGE/FGV indices, ANBIMA
  spreads) on deal activity. Sourced shortlist in the notebook; not wired in yet.
- **Phase 3 — Optional.** More public sources, light analysis automation, or a
  small dashboard.

## Disclaimer

Research and portfolio purposes only. Built from public information; the shipped
sample data is synthetic and labeled as such. Not investment advice, and nothing
here draws on any employer's or third party's proprietary or confidential data.
