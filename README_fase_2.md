# README — bloco da Fase 2

Substitua o item **Phase 2** do roadmap por isto:

---

- **Phase 2 — Credit × deals overlay.** *(built)* A per-sector **credit stress index**
  from one public source: **BCB SCR.data**, which publishes the corporate loan book and
  non-performing loans by CNAE section. Two signals are drawn from it — the current NPL
  **level** and its **12-month trend** — combined via z-score with the trend weighted
  heavier (0.6 vs 0.4). A deteriorating book is more informative than a high but stable
  one. The index is joined to deal activity and plotted as **activity × stress with
  quadrants**, surfacing the thesis corner: active sectors where credit is turning.

  The notebook rebuilds `stress_indicators.csv` straight from the BCB (set
  `REFRESH_STRESS = True`), stamping each run with the SHA-256 of the source archives.
  The derived CSV is committed, so a fresh clone runs offline with no download.

  **A note on the NPL definition.** BCB defines non-performance as the *full balance* of
  loans with any instalment more than 90 days overdue — not the overdue instalments
  alone. The obvious-looking field (`vencido_acima_de_90_dias`) is the latter and
  understates NPL by roughly half; the correct one is `carteira_inadimplencia`. The
  index cross-checks itself against the BCB headline: it refuses to write a file if
  aggregate corporate NPL lands outside a plausible band.

  **Why SCR.data and not SGS.** SGS — the BCB source most people reach for — has no
  sectoral corporate NPL series. Its corporate NPL series break down by loan *modality*
  (vehicles, cards, working capital), not by economic activity. The sectoral cut exists
  only in SCR.data.

---

E, na seção de outputs:

---

![Deal activity vs. credit stress by sector](figures/deals_vs_stress.png)

Reference period: March 2025 → March 2026. Aggregate corporate NPL: **2.78%**, up from
2.32% a year earlier. The mapping covers ~86% of the corporate loan book.

**Credit stress (Y) is real BCB data. Deal activity (X) is synthetic and labeled.** The
chart demonstrates the method of crossing the two — the position of any given bubble is
not a market call, because the deal side is generated. There is no public feed of
Brazilian mid-market transactions to draw on, and inventing one would defeat the point.

Read that way, the credit side does say something. **Retail** carries the worst NPL of
the mapped sectors (5.19%), **Education** the second (4.88%) and the steepest activity
in the synthetic set. **Agribusiness** is the interesting one: its level (2.91%) sits
below average, but it is deteriorating fastest (+1.05 pp in twelve months) — it lands in
the thesis quadrant only because the index weights trend over level. Weight the level
instead and it drops out. **Transport & Logistics** is the quiet counterpoint: high
stress, and (in this synthetic set) almost no deal flow.

---

## Fontes

- SCR.data — https://dadosabertos.bcb.gov.br/dataset/scr_data
- Metodologia (v2) — https://www.bcb.gov.br/pda/desig/metodologia_versao2.pdf

## Antes de commitar

```
echo ".scr_cache/" >> .gitignore
```

Os arquivos anuais do SCR somam ~250 MB e não têm nada que fazer no repositório.
