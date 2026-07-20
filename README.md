# Ad-hoc econ desk (`_desk`)

A single, long-lived Cursor workspace for **quick ticker lookups and small
calculations** — the kind that fill an `$XXX` blank in commentary — that don't
justify the full `newproj` scaffold.

## Use it

Open `_desk` in Cursor and keep it around. Describe the task (or paste
`econ-templates/prompts/adhoc_lookup.md`). `adhoc.mdc` is active here and
drives the light workflow: MCP-confirm tickers → pull + cache → show the calc
→ hand back the number in the output contract.

## Layout

- `.cursor/rules/` — `05-environment`, `10-data-sources`, `adhoc` (desk-only)
- `cache/` — raw Parquet pulls (gitignored)
- `tasks/` — dated notes per lookup (light audit trail, tracked)
- `sandbox.json` — network allowlist (Haver MCP / FRED / BBG / etc.)

## When to leave the desk

If a task grows real deliverables — a figure, a model, a newsletter — stop and
spin a real project with `newproj <name>`. The desk is for micro tasks only.

## Refresh

Rules and `sandbox.json` are copied from `econ-templates` at setup. To re-sync
after template changes:

    econ-templates/scripts/setup-adhoc-desk.sh --refresh-rules
