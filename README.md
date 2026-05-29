# Grover — Claude Code Plugin

A Claude Code plugin for Grover. This repository is **both** a plugin and a
[marketplace](https://docs.claude.com/en/docs/claude-code/plugins) that serves
it, so users can install it in two commands.

## Installation

```text
/plugin marketplace add will-stellar/grover-claude-plugin
/plugin install grover@grover-marketplace
```

On install you'll be prompted for your Grover **access token**; the plugin stores
it securely and uses it to authenticate the MCP server (see
[Authentication](#authentication)).

> **Supported on Claude Code (CLI) only.** The Claude Desktop app is **not yet
> supported** — see [Claude Desktop](#claude-desktop-not-yet-supported) below for
> why and the planned fix.

## What's included

| Component | Description |
|-----------|-------------|
| `hello` skill | Greets you as the "Grover Rig Helper" (run `/hello`). |
| `grover` MCP server | Connects to the remote Grover endpoint (`/mcp` on Vercel) with bearer-token auth. |

## Authentication

The Grover endpoint authenticates with a bearer **access token**. The plugin
declares it as a `userConfig` field (`accessToken`), so Claude Code:

- **prompts you for it at install time** — no manual file editing,
- stores it **securely** (`sensitive: true`, kept out of plaintext config), and
- **namespaces it per-plugin**, so it can't collide with other plugins' settings.

At install you'll see a prompt for **Access Token** — paste your `tok_...` value.
The plugin builds the header `Authorization: Bearer ${user_config.accessToken}`
from it. After installing, run `/mcp` — the `grover` server should show
**connected**.

To update the token later, reinstall the plugin (or run
`/plugin marketplace update grover-marketplace` and reinstall) to be re-prompted.

## Claude Desktop (not yet supported)

Claude Desktop **cannot** authenticate this server today, for two reasons:

1. **No `userConfig` install prompt.** Desktop doesn't surface the plugin's
   `userConfig` prompt, so the token is never collected.
2. **No `${user_config.accessToken}` interpolation.** That substitution is a
   CLI/SDK feature; in Desktop the placeholder is passed through literally, so the
   auth header is never filled in.

The result: in Desktop the plugin installs and the `hello` skill works, but the
`grover` MCP server loads without a valid token and never connects.

**The fix** is server-side, not in this plugin: add **MCP OAuth support** to the
Grover endpoint. Desktop supports remote MCP servers natively over OAuth, which
would let it connect directly with no token handling at all. Until then, use the
CLI.

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
