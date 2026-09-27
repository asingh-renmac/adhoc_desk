---
paths:
  - "plot*.py"
  - "*chart*.py"
  - "*_plot*.py"
  - "*fig*.py"
---
<!-- Generated from econ-templates/desk-skeleton/.cursor/rules/30-charts-desk.mdc by scripts/mdc_to_claude.py. Edit the .mdc, not this copy. -->

> File paths in this document are references, not attachments: read a referenced file before writing code that mirrors it.

# Charts at the desk: RenMac house style

Any chart made at the desk follows the same house style as a project. The
source of truth is the project rule and its helpers; this file only points at
them, so the style never forks.

- Rule: C:/Users/asingh/new_work/econ-templates/.cursor-rules-template/30-charts.mdc
- Style helpers: C:/Users/asingh/new_work/econ-templates/charts/renmac_chart_style.py
  (`renmac_style`, `C_PRIMARY` maroon, `C_SECONDARY` slate, `C_NAVY`,
  `PALETTE_EXTENDED`, source-line constants)
- Mixed-sign contribution bars: C:/Users/asingh/new_work/econ-templates/charts/stacked_contribution.py
  (`plot_stacked_contribution`; navy is reserved for the total line; asserts the
  stacks add up to the total)

Import them rather than copying:

    sys.path.insert(0, "C:/Users/asingh/new_work/econ-templates/charts")
    import renmac_chart_style as rc
    from stacked_contribution import plot_stacked_contribution

Desk specifics:

- Save to `outputs/figures/` as PNG at dpi=200, `bbox_inches="tight"`, with the
  plotted data alongside as CSV + Parquet.
- Title at most 11 words, sentence case, no colon; subtitle in parentheses.
- Daily bar charts: plot on a trading-day axis (consecutive slots, relabelled
  months) so weekends and holidays don't leave gaps between bars.
- A chart is a sign the task may be outgrowing the desk (see `adhoc`).
