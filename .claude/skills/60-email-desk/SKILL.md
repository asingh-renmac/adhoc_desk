---
name: 60-email-desk
description: "Sending (or reading) email from the desk via Microsoft Graph, and the house writing voice for anything drafted in the desk's name"
when_to_use: "Use when the user asks to draft, send or read an email from the desk, or to write commentary in the desk's name."
---
<!-- Generated from econ-templates/desk-skeleton/.cursor/rules/60-email-desk.mdc by scripts/mdc_to_claude.py. Edit the .mdc, not this copy. -->

> File paths in this document are references, not attachments: read a referenced file before writing code that mirrors it.

# Email and commentary from the desk

The desk sends only when the user asks for an email. Nothing is sent by
default, and a desk result is never pushed to a shared list.

## Writing voice

Anything written in the desk's name (commentary, an email body, a newsletter
paragraph) follows the shared voice reference, the same one the project
post-analysis prompts use:

- C:/Users/asingh/new_work/econ-templates/prompts/author_voice.md

Read it before drafting and check the draft against it before sending. The
hard rules that get checked mechanically: no em or en dashes anywhere, none of
its banned phrases, varied sentence length, paragraphs that open on the
verdict and close on a view rather than a recap. The corpus it was derived
from lives in `econ_commentary` (`config/author_voice.md` there is the
original measurement; the econ-templates file is the maintained version).
For a longer write-up, the project templates are the reference:
`econ-templates/prompts/post_analysis_commentary.md` and `post_analysis_email.md`.

## Sending: the helper

C:/Users/asingh/new_work/econ-templates/scripts/send_via_graph.py (read its
docstring; it encodes the gotchas).

1. Write a bundle to `outputs/email_<slug>/`: `email_draft.md` (body),
   `email_subject.txt` (one line), `email_attachments.txt` (one path per line,
   relative to the desk root).
2. Charts inline AND attached: list each chart twice in the attachments file,
   once with an `inline:` prefix and once plain. In the body, reference inline
   images by path with an explicit width, e.g.
   `<img src="outputs/figures/x.png" width="640" style="max-width:640px;width:100%;height:auto;">`
   (charts at dpi 200 are ~1,500px wide and Outlook shows native size).
3. Tables in the body: raw HTML tables with inline styles render reliably;
   markdown tables render without borders in Outlook.
4. Dry run first, from the desk root, and check the recipients line and that
   no image is flagged as unresolved:

       python /c/Users/asingh/new_work/econ-templates/scripts/send_via_graph.py \
           --subject-file outputs/email_<slug>/email_subject.txt \
           --body-file outputs/email_<slug>/email_draft.md \
           --attachments outputs/email_<slug>/email_attachments.txt --dry-run

5. Send with the same command without `--dry-run`. Recipients come from
   `PROJECT_EMAIL_RECIPIENTS` in the desk's `config/.env` (the user only,
   created by `setup-adhoc-desk.sh`). Widening that list, or passing a shared
   list such as `ECON_DAILY_RECIPIENTS`, needs the user's explicit say-so.
6. Limits: 3 MB per attachment; credentials are `AS_MSGRAPH_*` from
   `econ-templates/config/.env`. On failure, report the error; don't retry.

## Reading email

There is no shared reading helper in econ-templates. The working pattern is
C:/Users/asingh/new_work/2026_daily_chart_replicator/src/ingest.py: app-only
Graph token on the same `AS_MSGRAPH_*` app, server-side `$filter` on sender and
`receivedDateTime`, paging via `@odata.nextLink`, attachments fetched per
message. It needs the Mail.Read APPLICATION permission with admin consent
(403 otherwise). Adapt it for a one-off read; if reading becomes routine,
promote a general `read_via_graph.py` into `econ-templates/scripts/`.
