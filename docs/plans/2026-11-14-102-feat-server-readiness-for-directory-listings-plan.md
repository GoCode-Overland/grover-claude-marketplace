---
title: "feat: get the Grover MCP server ready for directory listings (annotations, role-filtered tools, server instructions)"
type: feat
status: proposed
date: 2026-11-14
issues: []
priority: P1
effort: M
repos: [grover-chat, grover-claude-marketplace]
---

# Server readiness for Claude and ChatGPT directory listings

Labels: VERIFIED = read in code or docs this session. INFERRED = deduced.

**This is a grover-chat change.** It lives here because it is the prerequisite for every
distribution step in plan 103, and this repo is where partners meet the server. The code units
belong in a grover-chat branch (dev first, `bun run check && bun run verify`, draft PR). It builds
on grover-chat plan `docs/plans/2026-08-17-003-chore-mcp-2026-07-28-audit-plan.md`, gap A5, which
found the same annotation gap when there were 54 tools.

## Problem

Partners connect to one server, `https://ops.getgrover.ai/mcp`, and see every tool Grover has,
including the ones only Grover staff may call. No tool says whether it reads or writes. Anthropic's
Connectors Directory requires both a `title` and a `readOnlyHint` or `destructiveHint` on every
tool; the developer portal flags tools without them (claude.com "Submit a connector to the
directory", fetched 2026-11-14). Claude also uses the read/write split to decide what to ask the
user before calling, so missing annotations make write tools feel the same as reads.

Who it hurts: every partner session (more tools in context, more wrong-tool calls, "Forbidden"
errors on tools they were never meant to see), and any directory submission (blocked until fixed).

## Evidence

- **128 tools registered** in `grover-chat/src/lib/public-mcp/tools/` (VERIFIED: 128 distinct names
  passed to `server.registerTool(` across the tool files).
- **Zero annotations.** No file under `src/lib/public-mcp/` contains `readOnlyHint` or
  `destructiveHint` (VERIFIED by grep). No tool sets a display `title`; the `title:` hits are input
  fields such as event titles (VERIFIED).
- **Staff-only tools are registered for everyone.** `registerTools`
  (`src/lib/public-mcp/tools/index.ts:382`) registers the proposal, billing, moderation,
  account-deletion, growth, ticker, partner-insert and pin-book tools unconditionally; each refuses a
  non-owner inside its handler (`denyNonOwner`, `server.ts:88`). Example:
  `create_proposal.ts:39-49` registers with the description "Grover-internal: draft a click-to-sign
  Order Form … Requires the grover_owner role." (VERIFIED). 57 tool files carry an owner-gate
  marker and 71 description strings say "Requires the grover_owner role" or "Grover-internal"
  (VERIFIED by grep; the exact count of owner-only tools is INFERRED at roughly a third of 128).
  A partner's tool list therefore advertises internal operations by name and description.
- **No server instructions, placeholder identity.** `buildPublicMcpServer` creates
  `new McpServer({ name: 'grover-public', version: '0.1.0' })` with no `instructions`
  (`server.ts:1211`, VERIFIED). Clients show the server name; instructions are where a server tells
  the model "call `list_companies` first and pass `companyId`".
- **OAuth works with DCR, not CIMD.** The authorization server advertises
  `registration_endpoint` and PKCE `S256` (`.well-known/oauth-authorization-server/+server.ts`,
  VERIFIED). The 2026-08-17 audit (gap A3) notes the current MCP spec prefers Client ID Metadata
  Documents and keeps DCR as deprecated-but-retained. Claude's directory accepts OAuth 2.0 and lets
  you hand Anthropic a client id and secret instead (developer portal step 6). Whether ChatGPT needs
  CIMD was not verified.

## Proposed change

1. Every tool gets a human `title` and annotations: `readOnlyHint: true` for `list_*`, `get_*`,
   `summarize_*`; `destructiveHint: true` for `delete_*`, `remove_*`, `void_*`, `revoke_*`,
   `cancel_*`, `retire_*`, `unlink_*`, `send_*`; `destructiveHint: false` with
   `idempotentHint` where true for `create_*`/`update_*`/`add_*`; `openWorldHint: false` throughout
   (the server touches only Grover's own data), except tools that fetch a URL
   (`add_kb_link`, `add_app_user_document_from_url`).
2. **Register by role.** Owner-only tools are registered only when `ctx.role === 'grover_owner'`
   (or the documented allowlist for the usage dashboard). The in-handler `denyNonOwner` stays as
   defence in depth. A partner's `tools/list` then contains only what their role can call.
3. **Server identity and instructions.** `name: 'grover'`, `version` from `package.json`, and a
   short `instructions` string: what Grover is, call `list_companies` first, most tools need a
   `companyId`, writes follow the caller's dashboard role, and where to get help.

## Implementation units (grover-chat)

1. **A single source of truth for annotations.** A table in `src/lib/public-mcp/toolAnnotations.ts`
   mapping every tool name to `{ title, readOnlyHint?, destructiveHint?, idempotentHint?,
   openWorldHint? }`, applied by a wrapper around `server.registerTool` in `buildPublicMcpServer`
   (the file already wraps `registerTool` three times, `server.ts:484,916,1173`). A vitest test fails
   if any registered tool is missing from the table. Verify: test passes; a `tools/list` call through
   MCP Inspector against a dev deployment shows titles and hints on all 128.
2. **Role-filtered registration.** Move the owner-only `register*` calls behind
   `if (isOwnerOrAllowlisted(ctx))` in `tools/index.ts`, reusing the existing name sets
   (`PIN_APPROVAL_TOOL_NAMES`, `PROPOSAL_TOOL_NAMES`, etc.). A test builds the server with a
   `company_admin` context and asserts none of those names appear. Verify: the test, plus a
   dev partner test account (per the test-account runbook) connected in Claude listing only its
   tools. This is authorization-adjacent, so give it an independent reviewer, as the workspace
   rules already require for `src/lib/modules/auth/**` changes (`public-mcp/` is not on that list;
   this is a recommendation, not a rule).
3. **Instructions and identity.** Set `name`, `version`, `instructions` in `buildPublicMcpServer`.
   Verify in MCP Inspector's server info pane.
4. **Smoke test the three clients** against dev: Claude Code (plugin), claude.ai custom connector,
   ChatGPT developer-mode connector. Record what each needed (CIMD or DCR, redirect URI) in
   grover-chat `docs/solutions/` so plan 103's submissions do not rediscover it.

## Risks and what would make this the wrong call

- Hiding tools changes behaviour for any staff account that is not `grover_owner` but has been
  calling a tool that then refuses it anyway; nothing reachable is lost (INFERRED from the
  in-handler gates).
- Wrong annotations are worse than none: a write marked read-only may be called without the user's
  confirmation. The table in unit 1 is reviewed by a person, tool by tool, not generated by prefix
  alone; the prefix rule is only the starting draft.
- MCP SDK v2 changes the `registerTool` handler signature (audit plan note). Land this on the
  current SDK; the wrapper keeps the migration to one place.

## Open questions for Will

1. Should `grover_admin` staff who are not owners see the owner-only tools (greyed) or not at all?
   Recommendation: not at all.
2. Display name in clients: "Grover" or "Grover for Partners"? The second leaves room for a rider
   connector later (plan 103, option E).

## Independent review (2026-11-14)

**Verdict: SOUND WITH CHANGES.** P1 is right. Effort is M–L, not M: 127 tools each need a title and three hints that a person has reviewed.

Checked:
- Claude's directory requires "a `title` and the applicable `readOnlyHint` or `destructiveHint`" on every tool. The portal groups tools by read and write and flags missing annotations (claude.com/docs/connectors/building/submission, fetched this session). VERIFIED.
- grover-chat has zero `readOnlyHint` or `destructiveHint` under `src/lib/public-mcp`. VERIFIED.

Corrections:
1. **Tool counts.**
   - The server has 128 distinct names, but `get_standout_users_by_circle` is commented out (`tools/index.ts:415`), so 127 tools are registered.
   - Staff-only tools number **63**, not "roughly a third": the owner-gated arrays in `tools/index.ts` total 62 (business-OS 24, billing 4, moderation 5, proposals 11, pin approvals 3, book sponsorship 2, onboarding 1, usage dashboard 1, link clicks 1, push 1, growth 2, account deletion 3, ticker 3, partner insert 1), plus `create_company` (`grover_admin`).
   - A partner therefore sees at most **64** tools after unit 2, not about 85 (plan 104 repeats the 85).
2. **ChatGPT's rule is stricter than Claude's.** OpenAI's app submission guidelines require every tool's `annotations` to set `readOnlyHint`, `destructiveHint` **and** `openWorldHint` as explicit booleans. The review form also asks for a justification of each value. Unit 1's table must set all three on every tool, not only "where applicable", or the ChatGPT scan fails.
3. **Unit 2 needs three gates, not one.**
   - grover_owner for the arrays above.
   - The `isInternalDashboardViewer` allowlist for `get_usage_dashboard`.
   - `grover_admin` for `create_company`.

   Reuse the exported arrays and add a test that every name in them is absent for a `company_admin` context. Do not also filter the company-role writes for `company_viewer` in this plan. They are gated by company role, so a viewer sees them refuse.
4. **The Unit 1 wrapper must be outermost-safe.** `server.ts` already wraps `registerTool` three times (audit, billing, access) and relies on install order. An annotation wrapper only edits the config object, so install it first and assert in a test that the config identity used by `withAccess` (a `WeakMap` keyed on the config object, `server.ts:1056-1074`) is preserved. Replacing the config object would silently drop every access declaration.
