# Grover — Claude Code Plugin

A Claude Code plugin for Grover. This repository is **both** a plugin and a
[marketplace](https://docs.claude.com/en/docs/claude-code/plugins) that serves
it, so users can install it in two commands.

## Installation

```text
/plugin marketplace add GoCode-Overland/grover-claude-marketplace
/plugin install grover@grover-marketplace
```

The first time you use the `grover` MCP server, your browser opens to sign in to
Grover (OAuth). After you approve, the connection completes automatically — there
is no token to paste or store (see [Authentication](#authentication)).

> **Works on both Claude Code (CLI) and Claude Desktop.** The server authenticates
> over the MCP OAuth standard, which both clients support natively.

## What's included

| Component | Description |
|-----------|-------------|
| `grover` MCP server | Connects to the Grover backend (`ops.getgrover.ai/mcp`) over OAuth2. |

## Authentication

The Grover endpoint authenticates with **OAuth2**, the
[MCP-standard auth flow](https://modelcontextprotocol.io/specification/draft/basic/authorization).
The plugin declares only the server URL — no token, header, or `userConfig`. When
the client first connects, the server returns `401` with a `WWW-Authenticate`
header pointing at its OAuth metadata, and the client takes it from there:

- **discovers** the authorization server from the endpoint's
  `/.well-known/oauth-protected-resource` metadata,
- **registers itself dynamically** (no pre-shared client ID),
- **runs the authorization-code + PKCE flow** in your browser, and
- **stores and refreshes the token itself** — nothing is kept in plugin config.

After installing, trigger the `grover` server (or run `/mcp`) — your browser opens
to sign in, and once approved the server shows **connected**. The token refreshes
automatically; there's nothing to rotate or re-enter.

This works the same on **Claude Code (CLI)** and **Claude Desktop**, since both
support remote MCP servers over OAuth natively.

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
```

## Development

Test changes locally without pushing to GitHub:

```text
/plugin marketplace add /path/to/grover-claude-marketplace
/plugin install grover@grover-marketplace
```

Validate the manifests:

```bash
claude plugin validate .
```
