# Ad-hoc econ desk (`_desk`)

Long-lived workspace for quick ticker lookups and small calculations. The
workflow is the always-on `adhoc` rule: micro-preamble, MCP-confirmed tickers,
pull and cache, explicit calculation, dated note in `tasks/`. No `plan.md`,
no approval gate.

## Once per session, before the first task

1. Verify the shell with the snippet in `05-environment`. Claude must have
   been launched from Git Bash in this folder.
2. Run the rule-freshness check described in `adhoc`.

## Skills here

- `10-data-sources` — pulling any series; Haver discovery through the
  haver-metadata MCP (`search_series` → `get_series`), which in Claude Code
  is the claude.ai connector named `Haver Metadata`.
- `60-email-desk` — drafting or sending email, in the shared author voice.
- `30-charts-desk` loads when a chart script is read; read it before writing
  the first one.
- `/adhoc` runs the desk workflow on the task typed after it.
- Team RenMac skills (`renmac-commentary`, `renmac-analysis-writeup`,
  `renmac-*charts*`, `renmac-titles-and-charts`) are user-only here: charts
  follow `30-charts-desk`, email and voice follow `60-email-desk`. Use one
  only when asked for it by name.

## Source of truth

`CLAUDE.md` and `.claude/` are copies stamped by
`econ-templates/scripts/setup-adhoc-desk.sh`. Change the original in
`econ-templates`, then `--refresh-rules`.
