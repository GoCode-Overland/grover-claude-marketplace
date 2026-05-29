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

## What's included

| Component | Description |
|-----------|-------------|
| `hello` skill | Greets you as the "Grover Rig Helper" (run `/hello`). |
| `grover` MCP server | Remote HTTP MCP server (`/mcp` on Vercel), bearer-token auth. |

## Authentication

The MCP server authenticates with `Authorization: Bearer ${GROVER_ACCESS_TOKEN}`,
interpolated from the environment. Set that variable once for your client:

### Claude Code (CLI)

Add it to `~/.claude/settings.json`:

```json
{
  "env": { "GROVER_ACCESS_TOKEN": "tok_your_token_here" }
}
```

(A shell `export GROVER_ACCESS_TOKEN=...` before launching `claude` also works.)

### Claude Desktop

Open **Settings → Customize** and add `GROVER_ACCESS_TOKEN` in the local
environment editor. Desktop injects it into the plugin's MCP server the same way.

> **Note:** `GROVER_ACCESS_TOKEN` lives in the shared, flat environment namespace
> (not per-plugin), so the `GROVER_` prefix is what keeps it from colliding with
> other plugins. The seamless alternative — no token handling on either client —
> is to add MCP OAuth support to the Grover server.

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
