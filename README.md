# Grover — Claude Code Plugin

A Claude Code plugin for Grover. This repository is **both** a plugin and a
[marketplace](https://docs.claude.com/en/docs/claude-code/plugins) that serves
it, so users can install it in two commands.

## Installation

```text
/plugin marketplace add will-stellar/grover-claude-plugin
/plugin install grover@grover-marketplace
```

The bundled MCP server reads your Grover **access token** from the
`GROVER_ACCESS_TOKEN` environment variable — set it once per client (see
[Authentication](#authentication)). This works on both Claude Code (CLI) and
the Claude Desktop app.

**Requirement:** Node.js (the bundled MCP server runs via `npx`).

## What's included

| Component | Description |
|-----------|-------------|
| `hello` skill | Greets you as the "Grover Rig Helper" (run `/hello`). |
| `grover` MCP server | Connects to the remote Grover endpoint (`/mcp` on Vercel) through a local `mcp-remote` bridge, with bearer-token auth. |

## Authentication

### Why a bridge?

The Grover endpoint is a **remote** MCP server that authenticates with a static
bearer token. Claude Code (CLI) can talk to it directly, but **Claude Desktop
cannot** — Desktop only supports OAuth for remote servers and has no way to send a
static `Authorization` header. To make one plugin work on both, the `grover`
server runs as a **local stdio process** (`npx mcp-remote …`) that proxies to the
remote endpoint and attaches the bearer header itself. Local stdio servers are
fully supported on both clients.

### Setting your token

Both clients build the auth header from `GROVER_ACCESS_TOKEN`. Set that variable
once for your client:

**Claude Code (CLI)** — add it to `~/.claude/settings.json`:

```json
{
  "env": { "GROVER_ACCESS_TOKEN": "tok_your_token_here" }
}
```

(A shell `export GROVER_ACCESS_TOKEN=...` before launching `claude` also works.)

**Claude Desktop** — open **Settings → Customize** and add `GROVER_ACCESS_TOKEN`
in the local environment editor, then restart Desktop.

The plugin's MCP config assembles `Bearer ${GROVER_ACCESS_TOKEN}` into an `env`
value (not into `args`) on purpose: Claude Desktop on Windows mangles spaces
inside `args`, so the space in `Bearer <token>` is kept out of the argument list.

> **Desktop fallback:** if the token isn't picked up (i.e. `GROVER_ACCESS_TOKEN`
> isn't interpolated by Desktop), set the fully-formed header directly instead —
> add `GROVER_AUTH_HEADER` = `Bearer tok_your_token_here` in the environment
> editor. `mcp-remote` substitutes that value with no interpolation by Claude.

> **Note:** these variables live in the shared, flat environment namespace (not
> per-plugin), so the `GROVER_` prefix is what keeps them from colliding with
> other plugins. The seamless long-term alternative — no token handling on either
> client — is to add MCP OAuth support to the Grover server.

## Uninstalling

```text
/plugin uninstall grover@grover-marketplace
/plugin marketplace remove grover-marketplace
```

## Repository layout

```text
.claude-plugin/
  plugin.json        # Plugin manifest (name, version, MCP server)
  marketplace.json   # Marketplace index pointing at this plugin (source: "./")
skills/
  hello/SKILL.md     # The hello skill
```

## Development

Test changes locally without pushing to GitHub:

```text
/plugin marketplace add /path/to/grover-claude-plugin
/plugin install grover@grover-marketplace
```

Validate the manifests:

```bash
claude plugin validate .
```
