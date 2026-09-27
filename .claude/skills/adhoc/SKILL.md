---
name: adhoc
description: "Run the desk's quick-task workflow on the task typed after the command: confirm tickers, pull, show the calculation, log it."
disable-model-invocation: true
---
<!-- Maintained in econ-templates/desk-skeleton/claude/skills/adhoc/SKILL.md. The workflow itself lives in the adhoc rule; edit that, not this. -->

Take the task the user typed after the command and run it through the desk
workflow in the `adhoc` rule, step by step: the two-line preamble, MCP-first
ticker confirmation, pull and cache, the explicit calculation, the dated note
in `tasks/`.

There is no fixed answer format: shape the result to the task. The
calculation, the series provenance and the caveats always travel with it.
