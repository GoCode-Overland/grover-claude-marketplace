---
title: "research: where Grover's MCP server should be distributed (Claude directory, plugin, ChatGPT, MCP Registry, a rider connector)"
type: research
status: proposed
date: 2026-11-14
issues: []
priority: P1
effort: M
---

# MCP distribution strategy

Labels: VERIFIED = read in code or official docs this session. INFERRED = deduced. Third-party
sources are named where used; prefer the official pages they point to.

## Problem

Grover ships its partner MCP server one way: a Claude Code plugin installed with two slash
commands from a GitHub repo. Partners are van builders, component makers and clubs. Their staff
are owners, marketing and support people (INFERRED from the company list and the copy kit's
audience), and few of them run Claude Code. The plugin is the developer-shaped door to a
non-developer audience, and nobody can find it unless Will sends the link.

## The options (researched 2026-11-14)

**A. Claude Connectors Directory (claude.ai/directory/manage).** Anthropic's directory of remote
MCP servers, browsable inside claude.ai, Claude Desktop and mobile. Official requirements
(claude.com "Submit a connector to the directory", VERIFIED): any paid Claude plan can submit;
HTTPS remote server; OAuth 2.0 for authenticated services; every tool has a `title` and a
`readOnlyHint` or `destructiveHint`; documentation URL, privacy policy URL, support contact, icon;
tested as a custom connector first. Submissions are scanned and listed as **Community** by default;
**Verified** is a later label. Anthropic can hold the OAuth client id and secret. If a plugin points
at the server, Anthropic asks you to submit the server as a connector anyway and pair them.
*Fit: best.* A partner types "Grover" in Claude's connector list and signs in. No URL, no CLI.

**B. Anthropic plugin directory.** The same portal accepts plugins. `claude plugin validate`
already runs the directory's checks locally (plan 101 found one blocker: no license). Plugins now
also install in claude.ai and Cowork, except plugins with a top-level `bin/` (code.claude.com
manifest reference, VERIFIED). *Fit: good only once the plugin carries something the connector
alone does not, i.e. skills (plan 104).* A plugin that is just a URL duplicates option A.

**C. ChatGPT apps.** OpenAI now has apps submitted and published as plugins; you submit the
production MCP server (`/mcp` URL), listing metadata, screenshots, reviewer access, and a domain
verification token on the server's domain; review covers quality and safety, and OAuth setup is a
common rejection cause (OpenAI app submission guidelines; details partly from third-party guides by
vendors, INFERRED). AllTrails, the closest outdoor-brand comparison, launched in ChatGPT in
November 2025 and announced dedicated apps in both Claude and ChatGPT in August 2026 (AllTrails
press releases). *Fit: good for partners who live in ChatGPT, which is likely many small-business
owners (INFERRED). Cost: a review cycle and a domain-verification file on `ops.getgrover.ai`.*

**D. Official MCP Registry (registry.modelcontextprotocol.io).** Metadata only: a `server.json`
with a `remotes` entry for the Streamable HTTP URL, published with `mcp-publisher` under a namespace
proven by GitHub org or DNS (modelcontextprotocol.io registry docs; publish steps partly from
third-party guides). Still a preview service. Clients such as VS Code read it. *Fit: low value for
partners, near-zero cost; do it for completeness.*

**E. A rider-facing connector (radical, separate product).** Everything above is the partner
server. AllTrails' move shows the other door: riders planning a trip in Claude or ChatGPT asking
"where can I camp near Moab with a 22-foot van?". A no-login, read-only Grover connector over
public data Grover already builds (federal land and MVUM layers, fire, smoke and snow overlays, the
official campground catalog in grover-data) would put Grover in front of riders at planning time,
with a link into the app. *Fit: strategic, not urgent. It is a new server surface in grover-chat
with its own rate limits and abuse model, and should use only sources Grover may republish.*

**Not recommended:** Cursor/VS Code-style developer marketplaces (wrong audience), and a ChatGPT
Custom GPT (superseded by apps).

## Recommendation and order

1. **Plan 102 first** (annotations, role-filtered tools, instructions). It blocks A and C.
2. **A: submit to the Claude Connectors Directory** as "Grover", Community listing, Anthropic-held
   OAuth client if DCR causes trouble. Update the README, the blog post and the copy kit to say
   "find Grover in Claude's connectors" instead of pasting a URL.
3. **C: ChatGPT**, after a developer-mode test against dev succeeds.
4. **B: plugin directory**, paired with the connector, once plan 104 adds skills worth installing.
5. **D: MCP Registry entry**, an afternoon, any time after 102.
6. **E: rider connector**, as its own grover-chat plan if Will wants it.

## Implementation units

1. **Prereqs** (plans 101, 102). Verify: `claude plugin validate . --profile directory` clean;
   MCP Inspector shows titles and hints on every tool for a partner test login on dev.
2. **Connector submission (A).** Prepare listing text (name, ≤200-char one-liner, ≤2,000-char
   description), docs URL (the blog post or a new `getgrover.ai/partners/claude` page), privacy URL,
   support contact, icon, use cases ("reads and writes"), test access for reviewers on a dedicated
   reviewer account (never a real partner's). Will submits in the portal (it needs his Claude
   account). Verify: portal status reaches listed; a fresh claude.ai account finds and connects it.
3. **ChatGPT (C).** Test in ChatGPT developer mode against `dev.getgrover.ai/mcp`; serve OpenAI's
   domain-verification token from `ops.getgrover.ai` (a grover-chat route); prepare screenshots;
   submit. Verify: the app appears for a test ChatGPT account and a partner flow (summarize chat
   sessions) works end to end on dev before prod.
4. **Registry (D).** `server.json` in this repo (`name: "ai.getgrover/grover"` via DNS proof, or
   `io.github.GoCode-Overland/grover`), `remotes: [{ type: "streamable-http", url:
   "https://ops.getgrover.ai/mcp" }]`; publish with `mcp-publisher`. Verify: the registry API
   returns the entry.
5. **Docs follow-through.** Update `README.md`, `grover-splash/content/blog/grover-mcp-connector-claude.md`
   and the partner copy kit with the directory path. Verify by reading the three side by side.

## Risks and what would make this the wrong call

- A public listing invites sign-in attempts from people with no Grover dashboard account. The
  sign-in page must say plainly "for Grover partners; riders use the app" and link to `/partners`
  (a grover-chat copy change in the OAuth HTML, `src/lib/modules/admin/oauth/html.ts`).
- Directory review may ask for changes to tool names or descriptions; budget a second pass.
- If partner usage of the connector is near zero (check `admin_mcp_logs`, which
  `installMcpAuditLogging` already writes, VERIFIED in the 2026-08-17 audit), the first move is a
  "connect Claude" step in partner onboarding, not more listings.

## Open questions for Will

1. Submit to Claude's directory under which account and organization name ("GoCode LLC", per the
   legal pages, or "Grover")?
2. Is ChatGPT worth a review cycle now, or after Claude's listing shows usage?
3. Do you want a rider-facing, no-login connector (option E) on the roadmap?
