<!--
Copyright © 2026 Genworks International
Portions Copyright © 2026 Gornskew Enterprises

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as
published by the Free Software Foundation, either version 3 of the
License, or (at your option) any later version.  Distributed WITHOUT
ANY WARRANTY; see <https://www.gnu.org/licenses/agpl-3.0.html>.
-->

# Readymacs: a Ready-to-Run Emacs for People and AI Agents

Readymacs is a complete, batteries-included GNU Emacs distribution
built for AI-assisted Lisp development. It runs three ways: as the
interactive **console** of a [Basalt](https://gitlab.genworks.com/genworks/basalt)
deployment, as a **standalone container**, or **directly on your
host** as a conventional Emacs configuration. In every mode it
carries a built-in MCP endpoint (the lisply backend) through which AI
agents — Claude Desktop, Claude Code, Cursor, Gemini CLI, Codex, LM
Studio, or anything else that speaks
[MCP](https://modelcontextprotocol.org) — can work alongside you in
the same running Emacs.

Readymacs is the Genworks-maintained fork of
[Readymax](https://github.com/gornskew/readymax).

![Readymacs Logo](img/skewed-colorful.png)

## Why Readymacs? The Inversion

The prevailing custom is to embed one AI agent *inside* the editor:
wired into a single application, speaking only through it, one more
fixture among the features. The editor is the agent's whole world.

Readymacs inverts that arrangement. The environment embeds no agent;
it *receives* them. Any MCP-capable client can connect from outside,
and the running Emacs — its buffers, its REPLs, its tooling — joins
the visiting agent's own toolkit: files opened, code evaluated,
builds run, at the visitor's initiative and under your supervision.

Nor is the reception unique to Emacs. In a Basalt deployment the
Gendl engine services answer the same lightweight HTTP protocol
(called *Lisply*), and offer a connecting agent the same reception —
each service in its own Lisp dialect.

This repository covers the Emacs environment itself. The wider
arrangement — whole service stacks started and stopped with one
command, every service agent-ready — lives with the
[Basalt](https://gitlab.genworks.com/genworks/basalt) build system.

## What Will I Find Here?

This repository holds two assets:

1.  the complete Emacs configuration — `dot-files/` — including the
    MCP (lisply) backend. Installed directly on a host, this is the
    whole product; no part of (2) is required.

2.  the build materials: a Dockerfile and scripts for casting the
    configuration as a container image, with the configuration
    pre-installed for the built-in `emacs-user` account.

Running that image alongside Gendl engine services and the rest of a
working deployment is a third thing with its own repository:
**[Basalt](https://gitlab.genworks.com/genworks/basalt)**. Basalt is
the deployment; **Readymacs** is the console and the image that
carries it.

In a deployment, the console's Docker compose service answers on the
network by its service hostname, while the container itself carries a
generated instance name assigned at startup. See the Basalt
documentation for the service roster and naming rules.

## The Three Installation Modes

**Mode A — In a Basalt Deployment (recommended):** clone the
[Basalt](https://gitlab.genworks.com/genworks/basalt) repository and
run `./basalt up` there. A whole deployment comes up around the
console: Emacs, Gendl engine services, monitoring.

That pulls and starts several Docker containers and leaves your host
machine untouched apart from shell convenience commands for reaching
the containerized Emacs (see the Basalt README). You do not need to
run `./setup`. You do not need Emacs installed on your host. You do
need Docker.

**Mode B — Standalone Container:** the image runs freestanding on
any machine with Docker — no deployment, no other services. From a
clone of this repository:

```bash
docker/run
```

The container comes up self-contained: the Emacs daemon running, the
MCP endpoint listening (host port 7081 by default; `-p` chooses
another), your `~/projects/` mounted at `/projects` when it exists.

**Mode C — Direct Host Installation:** run `./setup`. The Readymacs
configuration files are linked into your host account (`~/.emacs.d`,
`~/.bash_profile`, etc.) for use by your own host Emacs. This starts
no containers. MCP support is **off by default** in this mode: the
lisply-backend endpoints stay disabled until you enable them
(`./setup --with-mcp` or, from inside Emacs,
`M-x lisply-enable-host-server`). Two things vary in a host
installation: whether the endpoints are enabled, and whether an MCP
wrapper ([lisply-mcp](https://gitlab.genworks.com/genworks/lisply-mcp))
is configured in front of them. Understand that the endpoints, not
the wrapper, are the security boundary: enabled endpoints accept any
HTTP client that reaches them, wrapper or no wrapper, while a
configured wrapper with disabled endpoints admits nothing at all.
Read [docs/HOST_EMACS_MCP.md](docs/HOST_EMACS_MCP.md) first: on the
host this grants arbitrary code execution on your machine and is not
sandboxed the way the containerized modes are. Mode C only makes
sense if Emacs is already installed on your host.

**Any combination:** the modes are independent and each idempotent —
a host installation (`./setup`), a standalone container
(`docker/run`), and a full deployment (`./basalt up`) can all coexist
on one machine.

**Note:** `./setup` is meant for new Emacs installations where you
don't have, or don't mind replacing, a personal configuration. If
you are an experienced Emacs user with a preëxisting setup, run
`./setup --dry-run` to see what it would do without touching your
files, then wire your own init files into the standard Readymacs
ones.

## Features

### Native Emacs Config

- **The dashboard (`*dashboard*`)**, kept current by a background
    refresh process: your project directories and their freshness,
    the health of every service endpoint in the deployment, the
    day's org-mode agenda, and one-key entry into SLIME with any
    connected Lisp service. In a deployment, the banner reflects
    the console's service identity.

- **Preïnstalled, pre-native-compiled third-party packages** (examples):
  - [Slime](https://en.wikipedia.org/wiki/SLIME) for Common Lisp / Swank
  - Paredit-mode, Flycheck-mode, Company-mode
  - Magit, Org-mode
  - Doom Color Themes, theme switching functions

- **Lisply-MCP (Model Context Protocol) Elisp Backend** — the MCP
    service surface:
  - lets AI agents drive the running Emacs through standard
    [lisply-mcp](https://gitlab.genworks.com/genworks/lisply-mcp).
  - Defined & sideloaded locally from
    `dot-files/emacs.d/sideloaded/lisply-backend/`
  - See The MCP Configuration Surface below — this is a
    configuration surface worth understanding, not furniture.

- **Image builds**: the container image is built from
    `docker/Dockerfile` by `docker/build`, published to
    `genworks/readymacs` on Docker Hub (the upstream
    `gornskew/readymax` images remain drop-in compatible).

### The MCP Configuration Surface

The MCP layer is working gear, not decoration: its implementation is
public ([lisply-mcp](https://gitlab.genworks.com/genworks/lisply-mcp)),
it stands between connecting agents and your running Emacs, and you
should know what passes through it.

**What it does.** The wrapper speaks MCP to the client on one side
and plain HTTP to the backend on the other. The Emacs daemon answers
a small HTTP dialect on port 7080 in-container
(`/lisply/lisp-eval`, `/lisply/ping-lisp`, ...), and any service
speaking that same dialect gets the same treatment — which is why one
wrapper configuration serves the Emacs console and the Gendl engine
services alike, each in its own Lisp.

**The tools it presents to a connecting agent:**

| Tool | What it does |
|------|--------------|
| `lisp_eval` | evaluate code in the service's own Lisp — the working channel |
| `ping_lisp` | is anyone home |
| `get_docs` / `get_docs_list` | built-in documentation, served on demand |
| `http_request` | reach the service's HTTP endpoints through one gate |
| `lisply_search` | search the indexed document corpus (Readymacs consoles; `skewed_search` until 2026-09-09, still answered as an alias) |

**Where it gets its configuration.** In a deployment, `./basalt up`
generates the client registries (`mcp/claude_desktop_config.json` for
Claude Desktop, and the matching form for each bundled agent CLI). In
the standalone container the endpoint listens just as it does in a
deployment. On the host it works only where a lisply-mcp wrapper is
configured — and either way, the wrapper is reception, not the lock.
The lock is the **endpoints** themselves, which any HTTP client that
reaches them can call directly, no wrapper involved. In the
containerized modes that is fine — the container is the sandbox and
the endpoints open inside it. On the host it is exactly why they stay
disabled by default (see Mode C).

**What to understand before enabling it.** `lisp_eval` is arbitrary
code execution, by design. In a container, that is the point — the
container is the sandbox. On the host it is your machine; read
[docs/HOST_EMACS_MCP.md](docs/HOST_EMACS_MCP.md) first.

### Webshot — page captures from inside the container

`webshot URL [out.png] [WxH] [--mobile] [--settle=MS]` captures any
web page from a **real emulated viewport** — page JS and CSS both
see exactly the width you asked for, and `--mobile` adds touch
emulation, so phone-size captures are honest rather than merely
plausible. `webshot-clip URL SELECTOR [out.png]` clips to the first
element matching a CSS selector, rendering below-the-fold elements
fully. Every run gets a throwaway browser profile (no stale cache
while you iterate on a live page), 3D viewports render and appear
in the captures, and a virtual host resolves in-browser with
`--host-resolver-rules="MAP somehost container"`.

Webshot drives a headless browser: the default image variants carry
a lightweight headless shell for capture work; the workstation
variants carry a full browser with a GUI behind it. A lite console
can add the headless shell at runtime
(`M-x skewed-install` `headless-shell`).

### What Else Is Included

Beyond the Emacs daemon and the MCP layer, the image carries working
gear — every piece of it real and reachable:

- **The web terminal** (port 6942, answering to `webterm` from any
  shell in the container): how a person reaches the console through
  a web browser when no terminal is to hand. Agents connect through
  the MCP layer; people take the web terminal.

- **Webshot** (`webshot` / `webshot-clip`): page captures from
  inside the container — see the Webshot section above.

- **Bundled agent CLIs** (the `-aituis` variants, including
  `-full`): four terminal AI agents for conversing with an agent
  directly — a separate channel from the MCP endpoints, so you can
  talk with an agent in one window while it works the deployment's
  services through MCP. See Bundled Agent CLIs below.

- **Background processes**: the console runs its housekeeping as
  ordinary Emacs subprocesses — the dashboard refresher, file
  watchers, long-running builds — visible in the buffer list
  (`C-x b`), never wedging the editor.

- **`node`**: included for your own JavaScript work under
  `/projects` — builds and checks run inside the container rather
  than on your host.

- **`M-x skewed-install`**: capabilities added at runtime —
  on-demand fitting of the headless browser onto a lite console, or
  the agent CLIs onto any variant, without rebuilding the image.

## In a Basalt Deployment (recommended)

Everything runs inside Docker containers — **you need not run
`./setup`, install any configuration, or touch your own host
Emacs.** Your host machine stays clean apart from the shell
convenience commands `./basalt up` installs for reaching the
containerized Emacs.

### Requirements

 - Git
 - Docker — see [macOS-Specific Section](#macos-specific-section) if on a Mac

### Quickest Start — clone Basalt and start the deployment

```bash
git clone https://gitlab.genworks.com/genworks/basalt
cd basalt
./basalt up
```

Your `~/projects/` directory will become mounted at `/projects` in
the deployment's containers and will be created if missing.

Once an agent is connected, paste
[`docs/PROJECT_INSTRUCTIONS.md`](docs/PROJECT_INSTRUCTIONS.md) into a
Claude Desktop Project's custom instructions (or your `CLAUDE.md` /
`AGENTS.md`) as standing instructions for the session.

### Initial Setup (full clone)

1. Copy this repository anywhere you like — `~/readymacs` is fine:

```bash

   cd
   git clone https://gitlab.genworks.com/genworks/readymacs
   cd readymacs

```

   Cloning under your own `~/projects/` instead is useful only if you
   want to hack on Readymacs internals from inside the container
   (the host `~/projects/` directory is mounted at `/projects`
   there). For just *using* Readymacs to work on other projects,
   the clone location doesn't matter — the running container never
   needs the clone.

2. Start the default deployment (from a Basalt clone):

```
   ./basalt up

```

By default this pulls missing images only (no overwrites of local builds).
To force pulling the latest images, use:

```
   ./basalt up --pull
```

After the deployment is up, the generated shell convenience commands
for reaching the containerized Emacs are available in new shells on
your host — see the Basalt README for their names and usage, and for
what (single) modification is made to your shell startup files.

After you are in, see the "Getting Started" section near the top of
the default landing dashboard.

### Connecting an out-of-stack MCP client (Claude Desktop and friends)

This is trivial to do with the generated
`mcp/claude_desktop_config.json`. Please see [the Basalt
repository](https://gitlab.genworks.com/genworks/basalt) for details.

### Bundled Agent CLIs (Claude Code, Gemini CLI, Codex, Grok)

The `-aituis` image variants (including `-full`, which is an alias
for `gui-aituis`) bundle four terminal AI agents — launched from any
shell inside the container (`M-x vterm`) — while those same agents
reach the deployment's services through the MCP layer:

| Agent | Launcher | First login |
|-------|----------|-------------|
| Claude Code | `claudly` | OAuth URL to open in a browser |
| Gemini CLI | `geminly` | Google OAuth prompt |
| OpenAI Codex | `codexly` | Interactive login, or `OPENAI_API_KEY` |
| Grok Build (xAI) | `grokly` | `grok login`, or `GROK_DEPLOYMENT_KEY` |

They come up preconfigured — every service endpoint in the
deployment wired in: `./basalt up` merges the service configs and
installs them in whatever form each agent CLI expects, so an agent
you converse with in a terminal here reaches the same services an
outside Claude Desktop would. Credentials are volume-mounted from
your host and survive restarts and recreates.

A variant without them is not a dead end — `M-x skewed-install` fits
the agent CLIs on demand, though those fittings are ephemeral. And
an outside MCP client works identically against any variant, `lite`
included.

**Details** — which config lands where, why the launchers are shell
functions rather than binaries, the Grok credential-mount asymmetry,
and the build-stage layout — are in
[docker/README.md](docker/README.md).

## Windows-Specific Section

### Emacs-slanted Keyboard Tweaks for Windows

Readymacs uses the traditional Emacs keybindings by default, which
make heavy use of the Control key ("C-" in emacs parlance). For this
reason, it can be convenient to bind a more ergonomic key such as
CapsLock to Control, on modern keyboards. (Older keyboards had Control
in the place of current CapsLock). This repository
contains [instructions](windows-keybindings/README.md) for mapping
CapsLock to Control (with or without WSL) using a free program called
SharpKeys.

If you enjoy the traditional emacs keychords and want more of them in
your life, you can replicate those across most Windows programs using
the free AutoHotkey program, for which we bundle a config, also
described in the [instructions](windows-keybindings/README.md).

### Emacs in the Web Terminal (ttyd) on Windows

If you use the web terminal — Emacs in a browser tab on port 6942, or
a hosted session — Edge and Chrome will steal a few chords before the
terminal sees them: `C-n` opens a new window, `C-p` prints, and `C-w`
closes the tab you are working in. No setting inside the page can
stop that. We bundle a second AutoHotkey config,
`autohotkey-config-for-emacs-in-ttyd.ahk`, which catches those chords
at the OS level and hands the terminal something Emacs understands
(the shipped Emacs config binds `M-]` to `kill-region` for exactly
this reason). Run that script before you open a ttyd tab; the
[instructions](windows-keybindings/README.md#emacs-in-a-browser-tab-ttyd)
walk through it, plus an Edge registry policy for anyone who wants the
genuine control characters back.

## macOS-Specific Section

### macOS Prerequisites

`basalt` is pure POSIX sh — no special shell is required on macOS.
The only requirement is **Docker Desktop**.

#### Install Docker Desktop

Install [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/)
if you haven't already, then confirm:

```bash
docker info   # should print engine info without errors
```

Once Docker is running, `./basalt up` will work normally.

---

## Direct Host Installation

This section is for installing the Readymacs configuration
**directly on your host machine**, without Docker. It is independent
of the other modes — do not run `./setup` as part of a deployment or
standalone-container setup; it is not needed and not intended for
those.

1. Make a `~/projects/` directory if you don't already have one:

```bash

    cd
    mkdir -p projects/
    cd projects/

```

2. Clone this repo into `~/projects/`:

```bash

   git clone https://gitlab.genworks.com/genworks/readymacs
   cd readymacs

```

3. Run the setup script:
   ```bash

   cd ~/projects/readymacs
   ./setup

   ```

   The setup script will create symbolic links of the salient
   "dot-files" (hidden files starting with `.` pointing to the
   corresponding files in the cloned repo, for example:

    `~/.emacs.d -> ~/readymacs/dot-files/emacs.d`

   If you already have any of these dot files existing (as links or
   actual files/directories), the existing files will be backed up
   with names appended with `-pre-skewed-emacs`.

### Optional options for `setup`

- `--dry-run`: Shows what would happen without making any changes
- `--shadow-suffix=NAME` or `--shadow-suffix NAME`: Creates symlinks with a "-NAME" suffix
     (e.g., with `--shadow-suffix=test` or `--shadow-suffix test` creates ~/.emacs.d-test instead of ~/.emacs.d)
- `--scrub-shadow-suffix=NAME` or `--scrub-shadow-suffix NAME`: Removes all symlinks with the "-NAME" suffix
     (e.g., `--scrub-shadow-suffix=test` removes ~/.emacs.d-test, ~/.bash_profile-test, etc.)
- `--scrub-shadow-suffix=""` or `--scrub-shadow-suffix=`: Removes all default symlinks without a suffix (e.g., removes ~/.emacs.d, ~/.bash_profile, etc.)

The setup script will automatically detect and replace broken symlinks
and handle existing dotfiles by backing them up with a
`-pre-skewed-emacs` suffix. It also skips backup files ending with
tilde (~) in the dot-files directory.

####   Example with options:

```bash
   # Preview changes without modifying anything
   ./setup --dry-run

   # Install configuration files with regular names
   ./setup

   # Install configuration files with "-shadow" suffix
   # (useful for testing or for maintaining multiple configurations)
   ./setup --shadow-suffix=shadow

   # Install with a custom suffix
   ./setup --shadow-suffix=work

   # Preview shadow installation without making changes
   ./setup --dry-run --shadow-suffix=shadow

   # Preview custom suffix installation without making changes
   ./setup --dry-run --shadow-suffix=test

   # Remove all symlinks with the "-test" suffix
   ./setup --scrub-shadow-suffix=test

   # Preview removal of all symlinks with the "-shadow" suffix without making changes
   ./setup --dry-run --scrub-shadow-suffix=shadow

   # Remove all symlinks with the "-test" suffix and create new ones with "-work" suffix
   ./setup --scrub-shadow-suffix=test --shadow-suffix=work


```


⚠️ **Warning**: In case of malfunctions, the setup script may
               overwrite your existing `~/.emacs.d/` and
               `~/.bash_profile`. It is designed to back up this data,
               but it would still be wise to back up your existing dot
               files before running the `./setup` script.




## Terminal Icons Setup

Readymacs includes a flexible icon system for the dashboard and
org-mode agenda. By default we use colorful Unicode icons. If these do
not work in your terminal, or you'd like a more muted experience, we
recommend installing a **Nerd Font** in your terminal.

### Why Nerd Fonts?

With a Nerd Font installed, you can get flat professional looking
icons rather than loud colorful gaudy ones.

### Quick Setup

1. **Download a Nerd Font** from [nerdfonts.com](https://www.nerdfonts.com/font-downloads)
   - Popular choices: **Hack**, **FiraCode**, **JetBrainsMono**, **Meslo**
   - Download the "Nerd Font" version (not the regular font)

2. **Install the font** on your system:
   - **Windows**: Right-click the `.ttf` files → "Install"
   - **macOS**: Double-click the `.ttf` files → "Install Font"
   - **Linux**: Copy to `~/.local/share/fonts/` and run `fc-cache -fv`

3. **Configure your terminal** to use the Nerd Font:
   - **Windows Terminal**: Settings → Profiles → Defaults → Appearance → Font face
   - **iTerm2**: Preferences → Profiles → Text → Font
   - **GNOME Terminal**: Preferences → Profile → Custom font
   - **Alacritty**: Edit `font.normal.family` in config

4. **Enable nerd icons in Readymacs** by adding to your config or running:
   ```elisp
   (setq skewed-icons-style 'nerd)
   ```
   Or interactively: `M-x skewed-icons-set-style RET nerd RET`

### Available Icon Styles

| Style | Description | When to Use |
|-------|-------------|-------------|
| `ascii` | Pure ASCII characters | Dumb terminals, serial consoles |
| `unicode` | Safe geometric symbols | Default, works everywhere |
| `unicode-fancy` | Colorful Unicode + VS15 | Experimental, terminal support varies |
| `nerd` | Nerd Font icons | **Recommended** with Nerd Font installed |


### Troubleshooting Icons

- **Question marks in diamonds (�)**: Nerd Font not installed or not selected in terminal
- **Misaligned columns**: Switch from `unicode-fancy` to `unicode` or `nerd`
- **Icons look plain**: Install a Nerd Font and set `skewed-icons-style` to `'nerd`


## Configuration Structure

 - `dot-files/` - everything that ends up linked into your home
   directory when you run `./setup`
  - `emacs.d/` - Emacs configuration, to be linked to ~/.emacs.d/
    - `init.el` - Main Emacs configuration entry point
    - `etc/` - Modular configuration files
    - `sideloaded/` - Second-party packages
  - `bash_profile` - Bash configuration
  - `zshrc` - ZSH configuration

## Customization

For personal customizations that shouldn't be committed to this
repository, keep a file of your own — `~/.emacs-local` — read last
at every Emacs startup.

## License

AGPL-3.0-or-later, © 2026 Genworks International; portions © 2026
Gornskew Enterprises — see [LICENSE](LICENSE).
The vendored SLIME under `dot-files/emacs.d/sideloaded/slime-v2.28/` is
third-party and keeps its own terms; see [its
LOCAL-CHANGES.md](dot-files/emacs.d/sideloaded/slime-v2.28/LOCAL-CHANGES.md).
