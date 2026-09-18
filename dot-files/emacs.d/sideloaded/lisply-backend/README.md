# The Emacs Lisply backend — an HTTP endpoint for Emacs Lisp evaluation

This directory holds the Readymacs console's own **Lisply backend**:
a small HTTP service inside the running Emacs daemon that evaluates
Emacs Lisp on request and returns the result as JSON. The
[Lisply-MCP](https://github.com/genworks/lisply-mcp) middleware
connects to it and presents it to any MCP client — Claude Desktop,
Claude Code, Cursor, Gemini CLI, Codex, and the rest — as a set of
MCP tools; this directory is the server side of that arrangement.

The protocol is deliberately minimal: HTTP carrying JSON. Any
service that answers the same protocol — the Gendl engine services
in a Basalt deployment do — gets the same middleware and the same
tools. What a compliant backend must implement is specified in the
middleware's
[BACKEND-REQS.md](https://github.com/genworks/lisply-mcp/blob/devo/BACKEND-REQS.md).

## Endpoints

The backend listens on port 7080 inside the container (published as
7081 on the host when the container is started with `docker/run`;
see below). All paths sit under the `/lisply/` prefix:

| path | purpose |
|------|---------|
| `/lisply/ping-lisp` | availability check — answers `pong` |
| `/lisply/lisp-eval` | POST Emacs Lisp code; returns the result and captured standard output |
| `/lisply/tools/list` | the tools the middleware will expose: `ping_lisp`, `lisp_eval`, and `lisply_search` (when a search corpus is present) |
| `/lisply/lisply-search` | POST a query against the pre-built search index (see the Readymacs README, *The search index*) |
| `/lisply/docs/list`, `/lisply/docs/<id>` | documentation served on demand (`claude-md` is this backend's agent guidance; `main-claude-md` the repository's) |
| `/lisply/specs` | capability information for the middleware |
| `/lisply/resources/list`, `/lisply/prompts/list` | reserved; currently empty |

The prefix and the two main endpoint names are customizable
(`emacs-lisply-endpoint-prefix`, `emacs-lisply-ping-endpoint`,
`emacs-lisply-eval-endpoint`) to match a middleware configured with
different names; the defaults are what the middleware expects.

## Calling it directly

The middleware is the usual client, but any HTTP client works. From
a shell inside the container:

```bash
# availability
curl http://localhost:7080/lisply/ping-lisp

# evaluate an expression
curl -X POST http://localhost:7080/lisply/lisp-eval \
  -H "Content-Type: application/json" \
  -d '{"code": "(+ 1 2 3)"}'

# one that also prints to standard output
curl -X POST http://localhost:7080/lisply/lisp-eval \
  -H "Content-Type: application/json" \
  -d '{"code": "(progn (princ \"a message\") (* 6 7))"}'
```

From the host, with the standalone container running, use port 7081.
Never call the console's own endpoint from *inside* code the console
is evaluating: the HTTP server runs on the event loop that is busy
evaluating your request, and the call deadlocks.

## Responses

Every response is JSON. A successful evaluation returns

```json
{"success": true, "result": "6", "stdout": ""}
```

and a failed one returns

```json
{"success": false, "error": "the message"}
```

Results are rendered with `format "%s"`: strings keep their text,
lists their printed form, `t` and `nil` are themselves. There is no
interactive debugger on this endpoint — Emacs Lisp has no equivalent
of the Common Lisp restarts a Gendl service can offer — so an error
response is the whole story.

## Where it runs, mode by mode

- **In a Basalt deployment** (`./basalt up` in a Basalt clone):
  nothing to configure. The console starts with the endpoint listening
  on the deployment's network and the middleware already configured;
  `./basalt up` writes the MCP client configuration that points
  agents at it.

- **As a standalone container** (`docker/run` from a clone of this
  repository): the same, without the deployment — the endpoint listens
  inside the container and is published on host port 7081 (`-p`
  chooses another). The middleware is not in the container; point one
  at the published port:

  ```bash
  node /path/to/lisply-mcp/scripts/mcp-wrapper.js \
    --server-name readymacs --backend-host 127.0.0.1 --http-host-port 7081
  ```

- **Installed on your host** (`./setup`, the Readymacs configuration
  in your own Emacs): the endpoint is **off by default**, and for
  good reason. Read
  [docs/HOST_EMACS_MCP.md](../../../../docs/HOST_EMACS_MCP.md)
  before enabling it: on your own machine it grants arbitrary code
  execution with your user's privileges, and nothing sandboxes it.
  `M-x lisply-enable-host-server` enables it for one session after a
  warning you must acknowledge; `./setup --with-mcp` makes it
  permanent. Either way it binds to loopback only
  (`lisply-host-server-bind-address`).

> **Warning:** wherever it runs, `lisp_eval` is arbitrary code
> execution by design. In the container that is the point — the
> container is the sandbox, and nothing valuable is inside it unless
> you mount it. On a host it is your machine.

Loading the backend by hand, in any Emacs with `simple-httpd`
installed (`M-x package-install RET simple-httpd RET`):

```elisp
(add-to-list 'load-path "/path/to/lisply-backend/source/")
(load "http-setup")
(load "endpoints")
(emacs-lisply-start-server)   ; binds `httpd-host':`emacs-lisply-port' (7080)
```

`emacs-lisply-stop-server` stops it; `emacs-lisply-server-status`
reports which. In Readymacs none of this is typed by hand:
`etc/lisply-config.el` wraps it behind `lisply-enable-host-server`
and the warning.

## Files

- `source/http-setup.el` — the HTTP server: the listener, request
  and response plumbing
- `source/endpoints.el` — every endpoint above, and the pre-eval
  lint that refuses an unbounded child process (see `CLAUDE.md`,
  *the guard*)
- `source/lisply-shell-guard.el` — that guard, and
  `lisply-shell-bounded` / `lisply-shell-async`, the supported ways
  to run a subprocess from evaluated code
- `source/lisply-search.el`, `lisply-search-config.sexp` — the
  search index, and the list of sources an image build packs into it
- `source/lisply-edit-helpers.el`, `source/lisply-sexp-write.el` —
  helpers for agents editing files through the endpoint
- `CLAUDE.md` — the agent guidance served as the `claude-md`
  document: safe reading and editing in an Emacs shared with a
  person, the shared-buffer pitfall, paredit, the guard, and the
  search tool's parameters

## License

AGPL-3.0-or-later, © 2026 Genworks International, portions © 2026
Gornskew Enterprises — the same terms as the Readymacs repository
this directory belongs to, compatible with GNU Emacs's own GPL-3.0.
The AGPL adds one provision to the GPL: a modification used over a
network must be offered to the users it serves.
