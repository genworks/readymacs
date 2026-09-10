# Readymacs — agent guidance

This file guides Claude Code (claude.ai/code) or other AI agents
working in this repo.

## What this repo is

**Readymacs** is the Genworks-maintained fork of
[Readymax](https://github.com/gornskew/readymax) (remote `upstream`
in a working clone): a complete GNU Emacs distribution for
AI-assisted Lisp development, deployable as the console of a
[Basalt](https://gitlab.genworks.com/genworks/basalt) deployment, as
a standalone container (`docker/run`), or directly on a host
(`./setup`).

Upstream changes are merged when chosen; deliberate divergence is
limited to naming, attribution, and documentation voice.  Code fixes
that apply upstream belong upstream first.

## Register and naming rules for this fork

- Documentation in this repo is written in plain corporate voice —
  services, deployments, consoles, instance names.  No ship-and-crew
  vocabulary in prose.
- **Code identifiers are shared with upstream and are compatibility
  contracts — never rename them in a doc or naming sweep.**  That
  includes the `skewed-*` elisp namespace (`skewed-install`,
  `skewed-icons-*`, `skewed-dashboard-*`), the `lisply-*` backend
  namespace and endpoints, the `emacs-user` account, the
  `-pre-skewed-emacs` backup suffix, and the
  `~/.config/skewed-emacs/` config path.  Renaming any of these is a
  versioned behavior decision coordinated with upstream, never a
  register edit.

## Translation state (first light, 2026-09-01)

Translated to the Genworks register so far: `README.md`, this file,
`LICENSE` attribution, the dashboard footer attribution.

Still tracking upstream untranslated, on the catch-up list:

- `docker/build` and `docker/run` target `genworks/readymacs`
  (`IMAGE_REPO` overrides) and `.gitlab-ci.yml` runs on devo/master
  as of 2026-09-09; `BUILD.md` and `docker/README.md` still document
  the upstream `gornskew/readymax` coordinates.
- `docs/` and elisp docstrings/comments — upstream voice in places.
- The dashboard marquee art (READY MAX / READY ROOM guises in
  `dashboard-config.el`) — a Readymacs marquee needs its own art
  block before the standalone banner can change.

## Operational guidance

The deep operational material — MCP usage patterns, the shared-buffer
footgun, the event-loop/shell-guard rules, minibuffer-prompt hazards,
webshot/webshot-clip, magit-as-plumbing, bulk-edit verification —
lives in the upstream repo's `CLAUDE.md` and in
`dot-files/emacs.d/sideloaded/lisply-backend/CLAUDE.md` (present in
this repo).  All of it applies here as-is; where it names the
Basilisk stack or its service names, substitute the Basalt deployment
and its service roster.

Two rules worth restating because agents trip on them:

- Never run an unbounded synchronous child process through
  `lisp_eval` — use `(lisply-shell-bounded CMD &optional SECS)`; the
  backend guard refuses bare `shell-command` payloads.
- Never `find-file-noselect` project source files from batch evals
  (mode hooks can prompt and wedge the daemon); use
  `(with-temp-buffer (insert-file-contents ...))` for reads.
- Never curl the console's OWN lisply port from inside a
  `lisp_eval` (even bounded): curl waits on the httpd, and the
  httpd runs on the event loop that is blocked waiting for curl.
  Probe the console's endpoints with `lisply-shell-async`, from
  another service, or from the host.
