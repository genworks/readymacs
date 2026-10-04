# Standing Project Instructions for AI Clients

Durable instructions for an AI agent (Claude Desktop, Claude Code, Codex,
or any MCP-capable client) connected to a running Basalt deployment.

Where to put them:

- **Claude Desktop**: create a Project and paste everything below the
  horizontal rule into the Project's custom instructions.
- **Claude Code**: add it to your project's `CLAUDE.md`.
- **Codex**: add it to `AGENTS.md`.
- **Other clients**: wherever standing, every-session instructions live.

Prefer a one-shot first message instead of standing instructions? Use
[`mcp/opening-prompt.md`](https://github.com/genworks/basalt/blob/devo/mcp/opening-prompt.md)
from the Basalt clone — it walks a fresh session through the same
bootstrap interactively.

> Note: this repository's own `CLAUDE.md` is for working **on**
> Readymacs (development). This document is for **using** it. Keep
> them separate.

---

## At the start of each session

1. **Read the primer first.** Call the console's docs tool (`get_docs`
   with `id="primer"` on the Readymacs MCP server): short, and it
   covers reading, searching and editing workspace files — Lisp or not
   — through the console's `lisp_eval` rather than a shell.  Then
   evaluate `(lisply-help)` once to see the file helpers.
2. **Read the Dashboard** for deployment status, services, and
   available backends:

   ```elisp
   (with-current-buffer "*dashboard*" (buffer-string))
   ```

3. **Read the Daily Focus, if present** (org-mode agenda of
   Must/Should/Could priorities):

   ```elisp
   (progn
     (org-agenda nil "d")
     (with-current-buffer "*Org Agenda*" (buffer-string)))
   ```

   Daily Focus is optional. If it errors or is empty, the user hasn't
   set it up — skip it, and mention that `M-x skewed-daily-focus-init`
   creates a starter setup.
4. **Before using a Lisp backend**, read the `claude-md` docs of any
   backend you'll work with (the Gendl engine services, for example).
   The console's own longer docs (`claude-md`, `main-claude-md`) are
   references, for when the primer does not cover the case.
5. **Present options before diving in**: current state (which services
   are healthy), suggested next steps (from priorities/task notes), and
   any questions.

## Durable conventions (no doc re-read required)

### Shared-Emacs safety

You share one live Emacs — current buffer, point, and window state —
with an active human user.

- Read, search and edit files with the console's helpers —
  `lisply-read`, `lisply-grep`, `lisply-replace`, `lisply-form-replace`
  — not shell tools: they edit through a buffer the user has open
  instead of underneath it, refuse an edit that would unbalance a Lisp
  file, and never prompt.  A prompt in the console's Emacs stops every
  MCP call until someone answers it.
- Never open a project file with `find-file` or `find-file-noselect`
  from an eval (mode hooks can prompt); never bare `switch-to-buffer`.
- Target buffers explicitly, `(with-current-buffer BUF ...)`, and
  preserve point with `(save-excursion ...)`.
- Never assume the "current buffer" is yours.

### Paredit discipline (Lisp files)

- Prefer whole-form edits (`lisply-form-replace`, `lisply-form-insert`)
  and exact-text ones (`lisply-replace`); both check balance for you
  and write nothing if it would break.
- For finer structural work, use paredit in a temp buffer, then
  `(lisply-check-parens FILE)`.

### Discover backends from the Dashboard — never assume the set

The Dashboard's "Lisply Backends" section is the source of truth. A
standard Basalt deployment has three (the Readymacs console plus two
free Gendl engine services); overlays can add more. If it's unclear
which backend a task targets, check the task's notes or ask the user.

### Session state lives in org, not in static docs

If Daily Focus is set up, per-task context (`:HOST:`, `:NOTES:`,
LOGBOOK entries) lives in the org entries under `/projects/org/`
in-container (`~/projects/org/` on the host). Read it from there;
don't expect documentation to carry session-specific state.
