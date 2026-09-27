---
name: 10-data-sources
description: "How to pull macro data — use these reference scripts (vendor clients, scrapers, derived-series helpers)"
when_to_use: "Use before writing or running any code that pulls a data series (Haver, Bloomberg, Macrobond, FRED, BEA, BLS, Eurostat, UMich and the other reference scripts), or when choosing or confirming a ticker or Haver mnemonic."
---
<!-- Generated from econ-templates/.cursor-rules-template/10-data-sources.mdc by scripts/mdc_to_claude.py. Edit the .mdc, not this copy. -->

> File paths in this document are references, not attachments: read a referenced file before writing code that mirrors it.

# Data source conventions

The shared venv has Haver, xbbg, win32com (Macrobond), fredapi already.
Use them directly. Follow the patterns in these reference scripts:

Haver: C:/Users/asingh/new_work/econ-templates/data/haver_pull.py
Bloomberg: C:/Users/asingh/new_work/econ-templates/data/bbg_xbbg.py
Macrobond: C:/Users/asingh/new_work/econ-templates/data/macrobond_pull.py
FRED: C:/Users/asingh/new_work/econ-templates/data/fred_pull.py
BEA GDP-by-Industry: C:/Users/asingh/new_work/econ-templates/data/bea_pull.py
BLS (CES/CPS/P&C): C:/Users/asingh/new_work/econ-templates/data/bls_pull.py
Eurostat (public JSON-stat API, no key): C:/Users/asingh/new_work/econ-templates/data/eurostat_pull.py

## Haver — MANDATORY setup

Haver DLX 3.5.1 REQUIRES `Haver.direct("on")` before ANY Haver.data() call.
Auto path detection is unreliable and will fail the connection if you skip it.

- This call is already made ONCE at module load in haver_pull.py. Always go
  through that module (copy its setup block verbatim if adapting) — do NOT
  write a bare `import Haver; Haver.data(...)` connection test, which skips
  direct("on") and produces a spurious "connection failed" that costs a debug
  cycle. If you need to smoke-test, call haver_pull.test_haver_connection().
- If a Haver test fails, FIRST verify direct("on") ran, before assuming the
  install or entitlement is broken.

## Eurostat — public API, no key

C:/Users/asingh/new_work/econ-templates/data/eurostat_pull.py

- Plain HTTPS GET against the JSON-stat dissemination API — no key, no vendor
  DLL. Allowlist `ec.europa.eu` in sandbox.json for the project.
- The load-bearing gotcha (encoded): the response is a flat `value` dict keyed by
  a single integer offset over an N-dimensional cube. Decode it with row-major
  strides from `id`/`size`/category `index` — never assume dim ordering or a
  dense array. `_decode_jsonstat` does this for any dim count; use it, don't
  re-derive the indexing by hand.
- `s_adj` is a DIMENSION (NSA/SCA/SA/CA), not a flag — quarterly national
  accounts want SCA. Pin the geo you mean (`EU27_2020` = post-Brexit aggregate).
- `nace_a10_quarterly("va"|"hours", geo=...)` is the ready convenience for A*10
  real value added / hours worked; `eurostat_wide(...)` is the generic path.

## Scrapers and derived-series helpers (non-vendor sources)

These live alongside the vendor clients in data/ but cover sources that
aren't a straight vendor pull — a web scrape, a hybrid academic+scrape
series, or a derived chaining. Each is a REFERENCE: copy it into a
project's scripts/ and adapt the default paths, or import it if
econ-templates is on the path. Every gotcha below is already encoded in
the module's docstring — READ THE DOCSTRING before adapting one.

Mortgage rates (Mortgage News Daily):
C:/Users/asingh/new_work/econ-templates/data/mnd_pull.py
- The market-rate proxy when you need a daily 30Y fixed without a vendor
  entitlement. RenMac uses the "30 Yr. Fixed" series; resample("QS").mean()
  for a quarterly average. Other exposed series: 15 Yr. Fixed, 30 Yr. FHA,
  30 Yr. Jumbo, 7/6 SOFR.
- Fail-loud scraper: a network error, a layout change, an unparseable
  payload, OR stale data all raise MNDScrapeError — it never writes a
  bad/old file. assert_fresh() is the staleness backstop; wire it into any
  unattended refresh.
- Load-bearing gotchas (all encoded): pass the User-Agent (MND 403s bare
  GETs); balanced-brace JSON extractor, NOT a regex (chartData is nested);
  don't filter on 'DataChange' (silently drops points); tz-strip the date
  field (MND now serves ISO '…Z' strings → a tz-aware index breaks
  assert_fresh); atomic .tmp→replace write.
- On an unattended run, CONFIRM the live series label is exactly
  "30 Yr. Fixed" — a vendor-side rename hard-fails the pull by design.

Consumer Sentiment (UMich Surveys of Consumers):
C:/Users/asingh/new_work/econ-templates/data/umich_sca_scraper.py
- FINALS: FRED UMCSENT or Haver CSENT@USECON (1978→present). Use these
  directly — finals don't need the scraper.
- PRELIMS: there is NO native preliminary ticker (not in Haver, FRED, or
  ALFRED — vintages are end-of-month finals only). Don't hunt for one. Use
  the hybrid path: Kole et al. (2025) dataset for 1991-01→2021-09, UMich
  PDF scrape for 2021-10→present.
- Gotchas: the archive is the data subdomain
  (data.sca.isr.umich.edu/reports.php), NOT the homepage; polite scraping
  (3s sleep, identifiable UA, resumable progress log); two-pass PDF
  extraction (pdfplumber tables → narrative regex fallback) with an ICS
  sanity range [40,130]; the Kole 'All' sheet ≡ FRED UMCSENT, use it as a
  load-time cross-check; Kole citation REQUIRED in any published output
  (CC BY-NC 4.0). Implementation is lifted from the C-CSI project's
  scrapers at wrap-up — this file is the index to it.

Long-history Real PCE (chained 2017$):
C:/Users/asingh/new_work/econ-templates/data/real_pce_helper.py
- One call for Real PCE in chained 2017 dollars back to 1959. PCEC96 alone
  starts only 2007; this backward-chains DPCERA3M086SBEA (the quantity
  index, 1959→present) to PCEC96 via a single anchor (2007-01).
- If your transform is scale-invariant (log YoY, log diffs), you DON'T need
  this — the quantity index carries identical information. Reach for it only
  when you need levels/ratios in dollars pre-2007.
- Self-validates: backward-chained vs original PCEC96 correlation must be
  > 0.99999 over the overlap; a drop flags a possible BEA re-basing →
  re-anchor and validate manually. Never use it on a nominal series (PCE) —
  that's already current dollars, no chaining needed.

These three reference FRED/Haver IDs (UMCSENT, CSENT@USECON, PCEC96,
DPCERA3M086SBEA) are documented in the scripts above. If any should join
the trusted "Confirmed mnemonics" list below, that's a human-approval step
— don't auto-promote them.

Custom price aggregates (combine components, or exclude one from a parent):
use the INSTALLED `rmac-econ` package. Do not re-derive this arithmetic, and do
not copy it into `src/` — import it and pin the version.

    from rmac_econ.indexes import chain_aggregate, chain_subtract        # PCE
    from rmac_econ.indexes import laspeyres_aggregate, laspeyres_exclude  # CPI

    chain_aggregate(nominal=df, real=df, kind="fisher_of_fishers", ref_period="2017")
    chain_subtract(nominal=df, real=df, parent="services_ex_energy")
      -> ChainResult(quantity_growth, price_growth, price_index, quantity_index, nominal)
    laspeyres_aggregate(prices=df, weights=ri_df, base="2017-12", method="monthly")
    laspeyres_exclude(parent_price=s, excluded_prices=df,
                      parent_weight=w, excluded_weights=w, method="monthly")
      -> pd.Series (index level)

- NEVER add or subtract chain-weighted (Fisher) index levels. PCE is
  chain-weighted, so aggregation and exclusion both go through nominal + real.
  CPI's relative-importance form IS linear in weighted price relatives, so the
  Laspeyres pair may subtract — that licence does not transfer to PCE.
- The chain functions need nominal AND real (or `quantity=`) per component,
  never price levels alone. The CPI pair needs price levels plus BLS relative
  importances in parts per 100 (Haver `IPCU*` / `IUCX*`, monthly).
- Defaults are the accurate choice on both sides — CPI `method="monthly"`, PCE
  `kind="fisher_of_fishers"`. Do not override without a stated reason.
  `method="fixed"` is a constant-basket index, NOT a cheap approximation: it
  freezes the BLS weights at one December and can run ~30 bp hot on y/y, same
  sign every month. Never use `tornqvist_arithmetic` (legacy, biased −15.6 bp
  on y/y) for new work.
- Validated on every commit against six published Haver series: PCE exclusion to
  0.02 bp on m/m, CPI to ~0.3 bp with monthly re-anchoring. Method rankings and
  derivations are in the package's `docs/index_transforms_NOTES.md`.
- LEVELS are the weak spot, not growth rates. A Fisher index is not consistent
  in aggregation, so a reconstruction can drift >1% in level over decades while
  m/m stays inside 2 bp. Report growth rates from a reconstruction; justify any
  use of its level.
- The package's own test suite proves the LIBRARY, not your analysis. If this
  project needs a guarantee, replicate a published aggregate from its own
  components and gate on that in `40-validate` terms.
- If the import fails, the shared env predates the package: STOP and ask, do not
  vendor a copy. Source: https://github.com/renmac-econ/rmac_econ

## Component ticker catalogs — PCE and CPI (CHECK BEFORE RE-DERIVING)

Value-verified component/ticker inventories. If a project needs PCE or CPI
subcomponent tickers, weights, or a PCE↔CPI/PPI proxy map, START HERE — these
were expensive to build and every ticker is verified against the source, not
name-matched. Re-deriving them by MCP search is slower AND less reliable.

PCE: C:/Users/asingh/new_work/econ-templates/data/reference/pce/README.md
- `mb_pce_tickers_master.csv` — 151 core + 26 F&E nominal PCE tickers
  (Macrobond), bucketed Core Goods / Core Services / Food / Energy, with
  per-series start dates. Plus the 6 reference aggregates (core/total nominal
  denominators, core/headline price indexes, CPI context).
- `mb_to_haver_ticker_crosswalk.{csv,xlsx}` — the Macrobond→Haver port,
  183/183 value-verified, one unique Haver code per leaf. Carries `bea_code`
  (Macrobond's `BeaCode`) and `unit_scale_mb_over_haver`.
- `pce_cpi_ppi_proxy_crosswalk.csv` — the nowcast bridge: each PCE leaf's
  CPI/PPI proxy with `is_ppi`, `seasonal_status`, `sa_flag_x13`, `sign`.
- `market_based_pce_classification.csv` — the 13 non-market leaves + reasons
  (needed whenever you target a BEA *market-based* aggregate).
- `ismi_pce_panel_tickers.csv` — the 130-category Haver USNA panel (price /
  nominal / quantity) with core-vs-F&E classification.

CPI: C:/Users/asingh/new_work/econ-templates/data/reference/cpi/README.md
- Four component baskets → Macrobond tickers, each verified against the BLS
  supplemental table exactly (max relative error 0.00e+00), each with a
  `detail` sheet (BLS series ID, item code, relative importance, SA status).
- **Pick deliberately** — the four cut the CPI tree differently, and they are
  NOT interchangeable within one time series (switching mid-chart creates a
  break). `cpi_partition_*` (164, 99.4% of weight) is the DEFAULT: a true
  partition, mutually exclusive and near-exhaustive.
  `cpi_partition_max_granularity_*` (178, 97.0%) is that same partition
  descended deeper, buying components at the cost of ~3% leaked weight.
  `cpi_cleveland_fed_45` is the citable median-CPI set. `cpi_basket_*` (204) is
  the odd one out — NOT a partition and not exhaustive (20.8% of weight missing),
  with 66 footnote-(6) sub-sample rows carrying no relative importance: fine for
  an UNWEIGHTED diffusion count, WRONG for a weighted decomposition.
- These are a monthly BLS vintage (relative importances change) — re-run the
  upstream pipeline rather than hand-editing the workbooks.

Load-bearing gotchas from building these:

- **Haver carries NO BEA code field.** Macrobond exposes `BeaCode` on BEA-sourced
  series, but nothing on the Haver side joins to it — don't burn time trying.
  Match Macrobond↔Haver by VALUE (m/m %) instead.
- **Two gates, not one, when value-matching.** m/m correlation alone will happily
  match a CHILD to its PARENT aggregate (they correlate ~1.000). Require the
  level ratio to be clean too — exactly 1e6 for Macrobond raw USD vs Haver Mil.$,
  or 1.0 for indexes. An exact ratio plus identical m/m proves the same series.
- **Never filter the Haver PCE candidate pool by descriptor** — `cslem` is
  "Household Consumption Expenditures: Electricity" with no "PCE" in the name.
  Filter on Haver group `N68` (monthly US$ PCE nominals) instead.
- A flat CONSTANT series (e.g. BEA `DREERC0`) has m/m ≡ 0 and cannot be matched
  by correlation — verify it by a constant level ratio.

## Confirmed mnemonics (recorded so we don't re-browse DLX)

- Published NY/Dallas Fed WEI: FWEIW@WEEKLY
- Chicago Fed CARTS weekly retail trade: CARTSWRT@SURVEYS (SA, Bil.$);
  retail prices deflator: CARTSWRP@SURVEYS. Data starts 2018; ~single-week
  interior NaNs (the 4-weeks-per-month artifact) need filling.
- DTS daily withheld taxes: tdw@DAILY (current format; pre-2019 = tdw2..5,
  stitch). MTS withheld income: ftgriw@GOVFIN; employment: ftgrse@GOVFIN.
- Real GDP (quarterly, for WEI scaling): GDPH@USNA — NOT @USECON.
- NSA UI claims weekly: LICN@WEEKLY (initial), LIUN@WEEKLY (continuing).
- FOMC meeting shading (DAILY, +1=event / -1=none; 5-day shading windows):
  FOMC@DAILY (all meetings incl. unscheduled/conference calls); FOMCS@DAILY
  (scheduled only); FTARCHG@DAILY (funds-target rate change). Weekend
  meetings are remapped to the next business day (see Haver notes). Before
  joining market series, collapse each contiguous FOMC=+1 block to its
  LAST day (announcement proxy) — raw +1 days are not 1:1 with decision
  dates. Pull via pull_haver_daily() in haver_pull.py (never the monthly
  helper). Bloomberg alternative: BDH ECO_RELEASE_DT on FDTR Index
  (not FDTRMID) — see bbg_xbbg.py gotcha 12.
- FOMC funds target on announcement path: FFEDTR@DAILY (%). Step-filled
  daily level of the target (range midpoint when applicable); announcement
  days are where diff() ≠ 0 — use that to classify cut / hike / hold
  (e.g. 2008-10-29: 1.50→1.00 = −50bp cut). Complements FTARCHG@DAILY
  (+1/−1 change flag) when you need the size/sign of the move.
- PCE BVAR input panel (nine monthly series: headline/core PCE price, import
  price, PPI all-commodities, CPI all-items SA, core CPI, core-goods CPI,
  trade-weighted dollar, WTI) — value-matched against BEA/FRED in
  `2026_pce_bvar_mucsv_port`. Confirmed tickers + spans live in that project's
  `notes/haver_mnemonics.md` §4 (each matched BEA 51/51 or FRED exactly), NOT
  enumerated line-by-line here. Canonical mnemonics are in §1 of that file;
  the harvest text's inline names are informal. Port a ticker into this list
  only after re-vetting it individually.
- S&P 500 daily close: SP500@DAILY (1941-43=10, from 1955). Used for the B-S
  SP500_3M predictor (2026_rate_sensitive_sectors; matched the authors at 0.972).
- Bloomberg Commodity Spot Index daily: PZDJAS@DAILY (Jan-7-91=100, from
  1991-01-02). B-S BCOM_3M predictor (matched at 0.995); missing before 1991.
- Total nonfarm payrolls (SA, thous, monthly): LANAGRA@USECON (from 1939). B-S
  NFP_12M predictor (matched at 1.000).

## Discovering Haver mnemonics

To find or confirm a Haver series code, use the haver-metadata MCP
(search_series → get_series) instead of guessing or re-browsing DLX.
Confirm code@database + frequency + span via get_series before using
it. When you confirm a new mnemonic, SURFACE it for human approval
before it's added to the Confirmed mnemonics list above — don't write
it in automatically; that list is trusted and reused without
re-checking, so every line must be human-vetted. If the MCP isn't in
this workspace, ASK — never guess.

- For PCE or CPI COMPONENTS, check `data/reference/pce/` and
  `data/reference/cpi/` (above) before searching — those tickers are already
  resolved and value-verified, and a search will only re-derive them worse.
- Narrow search_series with the databases, frequency, geography, and
  sa_status filters when you know them — fewer, sharper candidates.
- Discontinued series are returned and LABELED (is_discontinued), not
  hidden; prefer a live series unless a discontinued one is explicitly
  wanted.
- Metadata only: the MCP finds and CONFIRMS series — it does NOT pull
  observations. After confirming the code@database ticker, fetch the
  data via the normal Haver path (the @haver_pull.py pattern above).

## Discovering Macrobond series

A `macrobond` MCP is also available (search + retrieval via the AI Data
Feed). Haver stays the DEFAULT for ticker discovery — free to query and
the confirmed-mnemonic list above is Haver-keyed. Use Macrobond when
Haver coverage falls short, or when the project already sits on
Macrobond tickers (the PCE/CPI reference sets above).

- Unlike haver-metadata, this one RETRIEVES data too, and fetches
  CONSUME UTS QUOTA. Preview first (`confirm=false`), show the user the
  series count and names, wait for explicit approval, then re-call with
  `confirm=true` and the returned invocation_token. Never skip the
  preview to save a round-trip.
- Call `get_instructions()` once per session before searching or
  fetching — it returns Macrobond's own discovery and presentation rules.
- Setup, and why the official Macrobond plugin CANNOT authenticate in
  Cursor: C:/Users/asingh/new_work/econ-templates/mcp/README.md. If the
  server shows `Incompatible auth server`, that's the known Cursor OAuth
  mismatch — the fix is the client-credentials proxy, not re-installing.

## Weekly index alignment (reusable pattern)

For any weekly multi-series index:
1. ONE canonical W-SAT grid (Sun–Sat reference weeks) spanning the sample.
2. Map each series to it with ONE documented fill rule (no ffill/bfill mix).
   W-FRI-dated series (fuel, electricity) → shift to the enclosing Saturday.
3. OUTER join, then an EXPLICIT logged NaN policy — never let a join
   silently drop weeks. Print per-series: first/last date, weeks present,
   NaNs filled, dropped.
4. 52-week diff = true 52 CONTIGUOUS-week lag on the grid, not .shift(52)
   on a ragged index. Verify grid contiguity over each window.

Mandatory patterns from the references:

- Save raw pulls to outputs/raw/ as Parquet IMMEDIATELY after fetching.
  Re-runs during development must hit the parquet, not re-pull.
- Scrapers (mnd_pull and friends) write a CANONICAL file to data/raw/ with a
  deterministic name (idempotent across runs) AND a per-run snapshot to
  outputs/raw/ as Parquet. NEVER put a timestamp in the canonical filename —
  that breaks re-runs; use a dated archive/ copy if you want history.
- Print first/last date + row count for every series after pulling.
- Tickers required by this project are in @prompt.md under "Data".
- If a Haver mnemonic, Bloomberg field, or Macrobond series ID is
  ambiguous, ASK — never guess.

## Cross-source gotchas (2026_pce_bvar_mucsv_port)

- **Haver sometimes carries far more history than FRED — check before building
  a splice.** FXTWBDI@USECON carries the Fed's trade-weighted-dollar
  back-estimate to 1973-03 while FRED's TWEXBGSMTH starts only 2006-01; a
  planned FRED splice turned out unnecessary. Pull the Haver span first.
- **Vendor gap-filling is per-series and undisclosed; never generalise from one
  series.** During the Oct-2025 shutdown, three CPI series from one vendor were
  filled three different ways (one an exact linear interpolation, another only
  ~38% of the way from a carry-forward). No single rule, no documentation.
  Prefer your own declared estimate even where the numbers happen to agree.
- **MCP/metadata spans are not authoritative — re-derive the endpoint from the
  pull.** All 89 series here reported a uniform `2026-04` from the metadata MCP
  while the actual DLX pull ran to `2026-06`; a uniform identical end_date
  across a large resolution set is the tell. Probe a handful of tickers and
  diff the realised endpoint against the catalog's claim before sizing a pull
  (the probe-before-pull helper in `data/`).

## Monetary-policy shocks (high-frequency and narrative)

C:/Users/asingh/new_work/econ-templates/data/mp_shocks.py
C:/Users/asingh/new_work/econ-templates/data/reference/monetary_policy_shocks/README.md

- START WITH THE README's strength table. Before building on any shock, compute the first-stage F
  (`first_stage_f`) at the frequency and on the rate you will use; rule of thumb 10. The FOMC-only
  Bauer-Swanson cleaned surprise is weak at quarterly frequency (F 0.7 on the 2-year, 1988-2019),
  and weak-instrument LP-IV output is noise, not a wider band around the right answer.
- Bauer-Swanson: authors' SF Fed file ends 2023-12. Rebuild/extend from USMPD with
  `bs_rebuild_per_meeting` (recipe + target correlations in bauer_swanson_rebuild.md). USMPD's own
  packaged surprise is RAW, not cleaned. Chair-speech surprises are not public.
- Narrative Romer-Romer: unified Bügel-Hidalgo-Luetticke series (quarterly 1969-2020; monthly
  file ends 2018-12) as the headline, Wieland-Yang (1969-2007) as the check. Strength is
  concentrated before 1983; say so when presenting, and don't restart the sample in 1990 to get a
  "modern" read (F 1.4-3.4, results are noise).
- Sources: www.frbsf.org, www.federalreserve.gov, raw.githubusercontent.com. Use
  `mp_shocks.download` (timeout, rejects HTML error pages).
