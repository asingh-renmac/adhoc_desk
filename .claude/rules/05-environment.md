<!-- Claude Code version of econ-templates/.cursor-rules-template/05-environment.mdc, maintained by hand in claude-template/overrides/05-environment.md because its Cursor text does not apply. Keep the two in step. -->

# Shell environment — Windows + Git Bash + shared venv

## What this rule guarantees

The shared Python venv at C:\Users\asingh\envs\shared-3.10 must be
active for any Python invocation, and the shared keys from
`econ-templates/config/.env` must be in the environment.

## How Claude Code gets the environment (read this once)

- **Shell.** With Git for Windows installed, Claude Code runs its Bash
  tool in Git Bash. It also offers a PowerShell tool. Use the **Bash
  tool** for every project command; never the PowerShell tool.
- **Venv and keys come from the launching terminal.** At session start
  Claude Code sources `~/.bashrc` only to capture aliases, functions
  and shell options; exported variables do not carry from one Bash
  command to the next. The venv (activated in `.bashrc`) and the keys
  (`.bashrc` sources `econ-templates/config/.env`) reach Claude only
  because they are already in the environment of the terminal that
  launched `claude`. So Claude must be started **from a Git Bash
  window, from the project root**:

      cd /c/Users/asingh/new_work/<project> && claude

  Started from PowerShell, Windows Terminal's default profile, or an
  IDE terminal that is not Git Bash, the session gets the wrong Python
  and no keys.
- **Billing.** `.bashrc` defines `alias claude='env -u ANTHROPIC_API_KEY
  claude'`. The API key in `config/.env` is for project scripts; without
  the alias it would switch Claude Code itself from the claude.ai Team
  seat to API-key billing. `claude auth status` should report
  `"authMethod": "claude.ai"`.

## Environment verification at session start

Before running any data pull or analysis, verify the shell with:

    uname -s && which python && \
      python -c "import os; print('FRED_KEY_OK=', \
      bool(os.environ.get('FRED_API_KEY')))" && \
      python -c "import os; print('MSGRAPH_OK=', \
      bool(os.environ.get('AS_MSGRAPH_CLIENT_SECRET')))"

Expected output:
MINGW64_NT-10.0-26200   (any MINGW64_NT-* means Git Bash)
/c/Users/asingh/envs/shared-3.10/Scripts/python
FRED_KEY_OK= True
MSGRAPH_OK= True

(The Cursor version prints `$0`; Claude Code's Bash tool stops for approval
on any command containing a `$` expansion, so this version avoids one.)

If shell shows anything PowerShell-related, you ran the check through
the PowerShell tool: re-run it with the Bash tool. If the Bash tool
itself is missing or is not Git Bash, STOP and tell Aman: Claude Code
did not find Git Bash, and the fix is to set
`"env": {"CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"}`
in `~/.claude/settings.json`, then restart Claude Code.

If python doesn't resolve to /c/Users/asingh/envs/shared-3.10/Scripts/python,
or FRED_KEY_OK / MSGRAPH_OK is False, STOP and tell Aman that Claude
was launched from a shell that had not run `~/.bashrc`: quit, open a
Git Bash window, `cd` to the project root, and run `claude` again.
Do NOT try to fix shell config from inside the agent loop — no
activating the venv per command, no sourcing `.env` by hand, no
editing `.bashrc`. These are user-level configuration issues.

## Env loading order — shared, then project-local

Two `.env` files, loaded in this order by any entry point that needs them
(`send_via_graph.py` already does this; mirror it in project scripts):

1. `econ-templates/config/.env` — shared credentials and shared lists
   (`AS_MSGRAPH_*`, `FRED_API_KEY`, `ECON_DAILY_RECIPIENTS`). Loaded with
   `override=False`, so a shell-set value still wins.
2. `<project_root>/config/.env` — this project's own vars, created by the
   scaffolder. Loaded with `override=True`: **project-local values win on
   collision.**

Project-specific vars (recipients, project flags) go in the project-local file
and are NEVER appended to `econ-templates/config/.env`, which stays reserved
for shared credentials. Scoping comes from the file's location, not from the
variable's name — so every project uses the same name,
`PROJECT_EMAIL_RECIPIENTS`, and the right value is picked up by running from
the project root. Both files are gitignored.

Because step 2 overrides, an exported shell value does NOT beat the
project-local file. That's deliberate: the project file is the scoping
mechanism.

## Path conventions in Git Bash

Git Bash translates /c/Users/... to C:\Users\... when passing as
COMMAND ARGUMENTS, but NOT when paths appear inside string literals
sent to `python -c`. Inside Python -c strings, use Windows-style
paths with forward slashes:

    # WRONG — Python sees the literal /c/Users/... which isn't valid
    python -c "import sys; sys.path.insert(0, '/c/Users/asingh/foo')"

    # RIGHT — Windows path with forward slashes (Python accepts both)
    python -c "import sys; sys.path.insert(0, 'C:/Users/asingh/foo')"

    # ALSO RIGHT — use cygpath to convert at command time
    python -c "import sys; sys.path.insert(0, r'$(cygpath -m /c/Users/asingh/foo)')"

The cleanest pattern: cd into the relevant directory and use
relative paths in -c strings.

## Anti-patterns

- DO NOT translate bash commands to PowerShell syntax, or switch to the
  PowerShell tool, to work around a failing command. Use the Bash tool.
- DO NOT pip install --upgrade to "fix" import issues. Verify which
  Python is running first.
- DO NOT `export` a variable in one Bash command and rely on it in the
  next: each command starts from the launch environment. Put what a
  script needs in the script, or in the project's `config/.env`.
- DO NOT silently re-run things in PowerShell because Get-ChildItem
  errors look like bash errors. The first sign that && or other
  bash tokens fail = PowerShell; use the Bash tool.
