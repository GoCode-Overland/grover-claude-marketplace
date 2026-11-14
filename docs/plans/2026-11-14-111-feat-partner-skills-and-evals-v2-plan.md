---
title: "feat (v2): partner jobs built only on partner-callable tools, shipped as MCP server prompts first and thin plugin skills second"
type: feat
status: proposed
date: 2026-11-14
issues: []
priority: P3
effort: M
supersedes: 2026-11-14-104-feat-partner-skills-and-evals-plan.md
repos: [grover-claude-marketplace, grover-chat]
---

# Partner skills and evals (v2)

Labels: VERIFIED = read in code this session (grover-chat checkout, 2026-11-14). INFERRED = deduced.

## What changed from v1

v1 (`2026-11-14-104-...`) proposed four plugin skills. The independent review found that one of them
depends on tools partners cannot call, and I confirmed it:

- `get_assistant_knowledge_sources` and `add_kb_text` are both in `BUSINESS_OS_TOOL_NAMES`
  (grover-chat `src/lib/public-mcp/tools/index.ts:141`, entries at `:158` and `:179`), which is gated to
  `grover_owner`. A partner running v1's `knowledge-base-gaps` skill would get "Forbidden", and once
  plan 102 hides staff-only tools they will not be listed at all. VERIFIED.
- Several writes need **company_owner**, not company_admin: `create_faq`/`update_faq`
  (`faqs.ts:214,335,413`) and `update_rig_template` (`update_rig_template.ts:28`) use
  `WRITE_OWNER_ROLES`. `summarize_chat_sessions` excludes `company_viewer` (`summarize_chat_sessions.ts:19,29`).
  VERIFIED.
- The tools the reworked jobs use (`list_companies`, `summarize_chat_sessions`, `list_knowledge_bases`,
  `get_kb_status`, `list_faqs`, `get_company_join_info`, `create_notification`, `get_rig_template`) are in
  none of the staff-only name lists in `index.ts` (grep of `:141-380`). VERIFIED.
- v1's "about 85 tools once staff-only ones are hidden" is wrong; the review counts at most 64. Not
  re-counted here, so this plan states no number.

And v1's second open question becomes the recommendation: a plugin skill reaches only clients that load
plugins, while an MCP **prompt** on the server reaches every client of the connector (Claude, ChatGPT,
the grover-admin dashboard assistant). The grover-chat MCP server registers no prompts today
(`buildPublicMcpServer`, `src/lib/public-mcp/server.ts:1210-1211`, and no `registerPrompt` anywhere in
`src/`; VERIFIED), so that is new work in grover-chat.

Priority drops to P3 (per the review): 2 of 30 partner companies made any MCP call in the last 30 days
(`get_usage_dashboard`, 2026-11-14). These jobs matter once plan 102 (staff tools hidden) ships and
connector use grows.

## The jobs, rebuilt on partner tools

Every job states the role it needs, never names a staff-only tool, and has one rule for writes: show
the exact change, ask, then call the write tool. If the tool answers "Forbidden: your role", explain
which role is needed and who in the company holds it; do not retry.

1. **Customer questions report** (role: company_admin or company_owner). `list_companies` →
   `summarize_chat_sessions` (30 days) → `list_knowledge_bases`. Output: top themes with the counts the
   tool returned, example questions, and three suggested actions. Read-only.
2. **Knowledge-base gaps** (role: company_admin; FAQ drafts need company_owner).
   `summarize_chat_sessions`, `list_knowledge_bases`, `get_kb_status`, `list_faqs`. Output: weak themes and,
   for each, a **suggested document** to add through the dashboard (partners cannot add KB text over MCP
   today), plus an FAQ draft. `create_faq` is called only after approval and only for company_owner.
3. **Circle launch** (role: company_admin). `get_company_join_info` for the join link and code, a pointer
   to the partner copy kit, and a first circle notification via `create_notification` only after approval.
4. **Rig template check** (read for company_admin; writes need company_owner). `get_rig_template`, list
   empty or implausible fields, propose `update_rig_template` changes, write only after approval.
   Coordinate field names with grover-chat's rig spec field ledger plan (`2026-11-14-252`).

No customer names, figures or internal SQL go into this public repo.

## Implementation units

**U1 (grover-chat, M): MCP prompt support and the first prompt.** Register prompts on
`buildPublicMcpServer`, starting with `customer-questions-report`. A prompt is text plus arguments
(company id, window), not new data access: it can only lead the client to call tools the caller is
already allowed to call. Tests: the prompt is listed; its text names no tool from any staff-only list in
`tools/index.ts` (a test that imports those arrays and checks, so a later rename cannot sneak one in).
Verify on dev with a partner test account from the test-account runbook.

**U2 (grover-chat, S each): prompts 2-4**, one PR each, same test.

**U3 (this repo, S): thin skills.** `skills/<name>/SKILL.md` for each job that says "use the Grover
`<name>` prompt" plus the role note and the ask-before-write rule, so plugin users get a discoverable
entry. Bump the plugin minor version.

**U4 (this repo, M): evals.** `evals/` with, per job: a happy case, a "no sessions in this window" case
(no invented themes), an "asks before writing" case that fails if a write tool is called without an
approval turn, and a "Forbidden: your role" case run as a company_viewer or company_admin (expects an
explanation, no retry). Run by hand against **dev** only. **Running them in this public repo's CI needs a
dev partner credential stored as a CI secret, a new long-lived secret on a public repo: needs Will's
explicit approval before it is set up.** Until then, evals are a manual pre-release step.

**U5 (this repo, S): README "What it can do for you"**, the four jobs in plain words, no tool names.

**U6: directory submission** of plugin plus connector (plan 103), after U1-U3.

## Risks and what would make this the wrong call

- If partners do not adopt the connector, none of this is used. U1's prompts also serve the grover-admin
  dashboard assistant, which partners do use, so start there.
- Tool names drift (`src/lib/mcp/legacyToolNames.ts` exists). The staff-list test in U1 and the evals in
  U4 are the guards.
- A prompt that suggests a write a partner's role cannot make produces a confusing refusal; the role note
  in each job and the Forbidden eval case cover it.

## Open questions for Will

1. Which of the four jobs do partners actually ask you for? Ship that prompt first.
2. Approve a dev partner credential as a CI secret on this public repo for scheduled evals, or keep evals
   manual?
