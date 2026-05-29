# Grover — Claude Code Plugin

A Claude Code plugin for Grover. This repository is **both** a plugin and a
[marketplace](https://docs.claude.com/en/docs/claude-code/plugins) that serves
it, so users can install it in two commands.

## Installation

```text
/plugin marketplace add will-stellar/grover-claude-plugin
/plugin install grover@grover-marketplace
```

On install you'll be prompted for your Grover **access token**, which the plugin
stores in your local Claude Code config.

## What's included

| Component | Description |
|-----------|-------------|
| `hello` skill | Greets you as the "Grover Rig Helper" (run `/hello`). |

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
