---
title: "fix: make the plugin directory-ready and write a README that gets a partner from zero to connected"
type: fix
status: proposed
date: 2026-11-14
issues: []
priority: P1
effort: S
---

# Directory-ready plugin metadata and a zero-to-connected README

Labels: VERIFIED = read in this repo, run locally, or read in grover-chat this session. INFERRED =
deduced.

## Problem

This repo is the public face of Grover's MCP connector: partners (builders, OEMs, clubs) install it
to manage their assistants, knowledge bases and circles from Claude. It works, but it reads as a
placeholder and it cannot be submitted to Anthropic's plugin directory as is.

Who it hurts: a partner's staff member who follows the link in the blog post
(`grover-splash/content/blog/grover-mcp-connector-claude.md`) and lands on a README that does not
say who can sign in, what to ask first, or what to do when it fails; and Will, if he wants the
plugin listed where partners can find it without a link.

## Evidence

- **Installable today.** `claude plugin validate .` (Claude Code 2.1.325, run 2026-11-14) ends in
  `√ Validation passed` (VERIFIED). The repo is public (VERIFIED, `gh api`), `marketplace.json`
  lists one plugin with `source: "./"`, and `plugin.json` declares one remote server,
  `https://ops.getgrover.ai/mcp`, type `http`. grover-chat serves RFC 9728 protected-resource
  metadata and an authorization server with dynamic client registration and PKCE
  (`grover-chat/src/routes/.well-known/oauth-protected-resource/+server.ts`,
  `…/oauth-authorization-server/+server.ts:12-22`, VERIFIED), which is what the README's
  Authentication section describes. An end-to-end sign-in was not run here (no partner account in
  this session).
- **Directory blockers from the same validator run** (VERIFIED):
  - `LICENSE_MISSING`: "This plugin has no LICENSE file and plugin.json has no `license`. The
    directory requires one of them." This is the one item the directory refuses a submission for.
  - `ICON_MISSING`: square PNG, 512-2048 px, under 2 MB; becomes the listing icon the first time the
    plugin is saved in the developer portal and cannot be changed later.
  - `PRIVACY_URL_MISSING`: a plugin that connects to a remote server needs `privacyPolicyUrl`.
- **Placeholder metadata.** `description` is "A plugin for Grover" in both `plugin.json` and
  `marketplace.json`; `author` is `{ "name": "Grover" }` with no url; no `displayName`, `homepage`,
  `repository`, `keywords`, `documentationUrl` or `supportUrl` (VERIFIED). The manifest reference
  lists all of these as supported fields (code.claude.com plugin manifest reference, fetched
  2026-11-14).
- **Version number with no policy.** `version: "4.0.2"` after 11 commits (VERIFIED, `git log`). The
  manifest reference says a set `version` "keeps users on that version until you change it", so a
  change to the server URL or a new skill reaches existing users only with a bump. Nothing records
  when to bump.
- **README gaps** (VERIFIED, `README.md`, 80 lines):
  - Says "Works on both Claude Code (CLI) and Claude Desktop" but gives only `/plugin` commands,
    which are Claude Code commands. The path most partners need, adding the server URL as a custom
    connector in claude.ai or the desktop app, is in the blog post, not here.
  - Does not say who can sign in. The server accepts Grover admin dashboard accounts, and what you
    can change follows your dashboard role; a viewer can read but not write (blog post, VERIFIED).
    A rider's app account cannot connect (INFERRED from the `admin_permissions` gate described in
    grover-chat AGENTS.md).
  - No first prompts, no troubleshooting ("Forbidden: your role cannot perform this action",
    re-auth with `/mcp`, browser not opening), no support contact, no privacy link.
  - Will's standing rule (grover-chat `docs/business-os-mcp-tools.md:8-10`, VERIFIED): this repo
    stays a light, high-level "how to connect" doc and does not enumerate tools. This plan keeps to
    that.
- **No check in CI.** There is no `.github/` directory (VERIFIED), so nothing runs
  `claude plugin validate` on a change.

## Proposed change and implementation units

1. **License.** Add `LICENSE` (MIT is the usual choice for a manifest-only repo; Will decides) and
   `"license"` in `plugin.json`. Verify: `claude plugin validate . --profile directory` no longer
   reports `LICENSE_MISSING`.
2. **Metadata.** In `plugin.json`: `displayName: "Grover"`, a real description ("Manage your Grover
   assistants, knowledge bases, circles and customer questions from Claude."), `author` with
   `url: https://getgrover.ai` and a support email, `homepage`
   `https://getgrover.ai/blog/grover-mcp-connector-claude`, `repository`, `keywords`
   (`grover`, `camping`, `rv`, `overland`, `customer-support`, `knowledge-base`),
   `privacyPolicyUrl: https://getgrover.ai/legal/privacy`, `termsOfServiceUrl:
   https://getgrover.ai/legal/terms`, `documentationUrl`, `supportUrl`. Same description in
   `marketplace.json`. Verify: validator passes with zero warnings on Claude Code ≥ 2.1.281 (older
   versions warn on the directory fields, per the reference).
3. **Icon.** `.claude-plugin/icon.png`, 1024x1024, the Grover knot from the public brand kit
   (`getgrover.ai/brand`, symbol on a linen plate). Logo use needs Will's OK (workspace brand rule).
   Verify: validator stops reporting `ICON_MISSING`.
4. **Version policy.** Add a "Versioning" section to the README: bump minor for a new skill or a
   server URL change, patch for docs; keep `version` in `plugin.json` only (the manifest's value wins
   over the marketplace entry's). Reset is not needed; continue from 4.0.2. Tag releases `v4.1.0`
   etc. Verify: the next PR that touches `plugin.json` bumps the version.
5. **README rewrite**, in this order, plain and short:
   1. What it is (one paragraph) and who can use it (people with a Grover partner dashboard login;
      what a viewer vs. an admin can do).
   2. Connect, three paths: Claude Code plugin (two commands); claude.ai and Claude Desktop custom
      connector (Settings → Connectors → Add custom connector → paste `https://ops.getgrover.ai/mcp`);
      once listed, "find Grover in the Connectors directory" (plan 103).
   3. First five things to ask (no tool names): "Summarize what my customers asked last month",
      "Which knowledge bases does my <assistant> use?", "Add this PDF to my manuals knowledge base",
      "Draft a notification to my circle about <event>", "What's in my rig template for <model>?".
   4. Troubleshooting: sign-in loop, "Forbidden" (your role), wrong company, how to disconnect.
   5. Privacy and support links; uninstall.
   Verify: a person who has never seen Grover's MCP follows it on a fresh Claude account with a
   partner test login on **dev** (per the workspace test-account rule) and connects without help;
   note where they stalled.
6. **CI.** `.github/workflows/validate.yml` that installs Claude Code from npm and runs
   `claude plugin validate . --profile directory` on pull requests. Verify: a PR that removes the
   license fails the check.

## Risks and what would make this the wrong call

- Listing the plugin in Anthropic's directory with only an MCP server adds little over listing the
  server as a connector (plan 103). The metadata work is still worth it: it is small, and plan 104's
  skills make the plugin worth listing.
- The README's claude.ai path points partners at production (`ops.getgrover.ai`). That is correct
  for partners; internal testing must use `dev.getgrover.ai/mcp` with a dev account, which belongs
  in a "Development" note, not the main path.

## Open questions for Will

1. License: MIT, or "all rights reserved" (the directory accepts any SPDX id or a LICENSE file)?
2. OK to use the knot mark as the plugin and connector icon?
3. Which support address should partners use (will@ or a shared support@)?

## Independent review (2026-11-14)

**Verdict: SOUND.** P1 and effort S are right.

Re-ran `claude plugin validate .` (2.1.325): it passes with two directory warnings, `ICON_MISSING` and `PRIVACY_URL_MISSING`. With `--profile directory` it fails on `LICENSE_MISSING` alone. That matches the plan exactly. The manifests are at `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`, and both have `description: "A plugin for Grover"`.

Corrections:
1. `privacyPolicyUrl` should point at a policy that covers the MCP/dashboard data flow: which admin and partner data the server reads, and the conversation logs (`admin_mcp_logs`). If `/legal/privacy` is the marketing-site policy, a partner-facing section is needed. OpenAI's app review also requires a policy that lists data categories, recipients and retention (developers.openai.com/apps-sdk/app-submission-guidelines). Check its content, not just that the link exists.
2. README "who can sign in": sign-in to the MCP server is an email magic link to the dashboard account (grover-chat `src/lib/modules/admin/oauth/html.ts:8,69`). Say so, and say that the link must be opened in the same browser. That is the most likely "sign-in loop".
3. Unit 6's CI must install Claude Code with a pinned version. The validator's checks change between releases, and an unpinned install will turn the check red on its own.
