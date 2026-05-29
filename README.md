# Grover — Claude Code Plugin

A Claude Code plugin for Grover. This repository is **both** a plugin and a
[marketplace](https://docs.claude.com/en/docs/claude-code/plugins) that serves
it, so users can install it in two commands.

## Installation

```text
/plugin marketplace add will-stellar/grover-claude-plugin
/plugin install grover@grover-marketplace
```

The plugin bundles the Grover MCP server, which authenticates with a **bearer
access token**. Supply it via the `GROVER_ACCESS_TOKEN` environment variable
(see [Authentication](#authentication)).

## What's included

| Component | Description |
|-----------|-------------|
| `hello` skill | Greets you as the "Grover Rig Helper" (run `/hello`). |
| `grover` MCP server | Remote HTTP MCP server (`/mcp` on Vercel), bearer-token auth. |

## Authentication

The bundled MCP server reads the token from the `GROVER_ACCESS_TOKEN` environment
variable. Set it once in `~/.claude/settings.json`:

```json
{
  "env": { "GROVER_ACCESS_TOKEN": "tok_your_token_here" }
}
```

Claude Code injects this into the session, so the `${GROVER_ACCESS_TOKEN}`
placeholder in the plugin's MCP config resolves automatically. (A shell
`export GROVER_ACCESS_TOKEN=...` before launching `claude` also works.)

## Uninstalling

```text
/plugin uninstall grover@grover-marketplace
/plugin marketplace remove grover-marketplace
```

## Repository layout

```text
.claude-plugin/
  plugin.json        # Plugin manifest (name, version, user config)
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
