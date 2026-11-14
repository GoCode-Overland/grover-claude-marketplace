---
title: "feat: partner skills (customer-questions report, knowledge-base gaps, circle launch) with evals"
type: feat
status: proposed
date: 2026-11-14
issues: []
priority: P2
effort: M
---

# Partner skills and evals

Labels: VERIFIED = read in code, on GitHub or in official docs this session. INFERRED = deduced.

## Problem

The plugin carries one thing, a server URL (`.claude-plugin/plugin.json`, VERIFIED). A partner who
installs it gets 128 tools today (about 85 once plan 102 hides staff-only ones, INFERRED) and a blank
prompt. The blog post says the summary tool
"is the one partners tend to underestimate" and suggests asking for a 30-day summary as the first
test (VERIFIED, `grover-splash/content/blog/grover-mcp-connector-claude.md`). That advice lives in
a blog post; nothing in the product turns it into a repeatable job. Skills are how a plugin packages
"do this job well, the same way every time", and the plugin directory treats a plugin with skills
as worth listing separately from the bare connector (plan 103, option B).

History: PR #1 proposed a `usage-dashboard` skill for Grover staff. Will closed it on 2026-10-11:
"this plugin and repo are public and go to partners, so an internal-staff skill doesn't belong
here" (VERIFIED on GitHub). The skills below are partner jobs only.

## Proposed skills

Each is a `skills/<name>/SKILL.md` that names the job, the tools to use, the order, the output
format, and what not to do. No customer names, figures or internal SQL in this public repo.

1. **`customer-questions-report`**: "What did my customers ask this month, and what should we fix?"
   Uses `list_companies` → `summarize_chat_sessions` (30 days) → `list_knowledge_bases`. Output: top
   themes with counts as returned, example questions, and for each theme whether a knowledge base
   seems to cover it, ending with three suggested actions (add a doc, write an FAQ, post to the
   circle). Never invents counts; quotes only what the tool returned.
2. **`knowledge-base-gaps`**: "Which questions is my assistant weakest on, and what document would
   fix each?" Uses `summarize_chat_sessions`, `get_assistant_knowledge_sources`, `get_kb_status`,
   `list_faqs`; proposes `add_kb_text`/`create_faq` drafts and asks before writing anything.
3. **`circle-launch`**: "Help me launch Grover to my customers." Uses `get_company_join_info` for the
   join link and circle code, points to `getgrover.ai/partners/copy-kit` for the email series, and
   drafts a first circle notification with `create_notification` only after explicit approval.
4. **`rig-template-check`**: "Is my rig template complete for the <model>?" Uses
   `get_rig_template`, lists empty or suspicious fields, proposes `update_rig_template` changes and
   asks before writing.

All write steps follow one rule stated in every skill: show the exact change, ask, then call the
write tool. Plan 102's `destructiveHint` makes Claude ask as well; the skill should not depend on
that alone.

## Implementation units

1. **Skill 1 and the eval harness.** Add `skills/customer-questions-report/SKILL.md` and an
   `evals/` directory (the manifest's default; `experimental.evals` in `plugin.json` only if a
   different path is used, per the manifest reference). Two eval cases: a company with chat sessions
   (expects themes and three actions, no invented numbers) and a company with none (expects a plain
   "no sessions in this window" and no fabricated themes). Run with `claude plugin eval` against
   **dev** using a dev partner test account from the test-account runbook; never production data
   (evals send real tool calls). Verify: both cases pass three runs in a row; bump plugin to 4.1.0.
2. **Skills 2-4**, one PR each, each with at least one "asks before writing" eval case that fails if
   a write tool is called without an approval turn. Verify: evals pass on dev.
3. **README section "What it can do for you"** listing the four jobs in plain words (no tool names,
   honouring the light-README rule). Verify: plan 101's fresh-reader test.
4. **Directory submission** of the plugin paired with the connector (plan 103 step 4). Verify:
   `claude plugin validate . --profile directory` clean; portal status.

## Risks and what would make this the wrong call

- Skills reach only clients that load plugins (Claude Code, claude.ai and Cowork per the manifest
  reference); a partner who adds the bare connector or uses ChatGPT gets none of this. If most
  partners connect that way, the same guidance belongs in the server's `instructions` and in MCP
  prompts on the server (grover-chat), not in skills. Check connector usage first (plan 103 risk).
- Tool names drift in grover-chat (`legacyToolNames.ts` exists, VERIFIED in `src/lib/mcp/`). A skill
  that names a renamed tool silently degrades. The evals in units 1-2 are the guard; run them in this
  repo's CI weekly against dev.

## Open questions for Will

1. Which of the four jobs do partners actually ask you for? Ship that one first.
2. Should these jobs also exist as MCP prompts on the server, so ChatGPT and bare-connector users get
   them too? That is the more universal home; skills can then stay thin.

## Independent review (2026-11-14)

**Verdict: NEEDS REWORK.** Two of the four skills call tools partners cannot use. Lower the priority to P3 until plan 102 ships and connector usage rises above 2 of 30 companies.

Corrections:
1. **Skill 2 (`knowledge-base-gaps`) calls `grover_owner`-only tools.** `get_assistant_knowledge_sources` and `add_kb_text` are both in `BUSINESS_OS_TOOL_NAMES` (grover-chat `src/lib/public-mcp/tools/index.ts`, the knowledge-curation and ingestion entries). Every partner would get "Forbidden". After plan 102 unit 2 those tools will not even be listed. Rebuild it on partner tools only: `summarize_chat_sessions`, `list_knowledge_bases`, `get_kb_status`, `list_faqs`. The output should be a document *suggestion*, since partners cannot add KB text over MCP today.
2. **Several writes need company_owner, not company_admin.** `create_faq`/`update_faq` (`faqs.ts:214,335,413`) and `update_rig_template` (`update_rig_template.ts:28`) use `WRITE_OWNER_ROLES`. `summarize_chat_sessions` excludes `company_viewer`. Each skill should state the role it needs. Each skill also needs an eval case for the "Forbidden: your role" path, so the skill explains it instead of retrying.
3. **"About 85 tools once staff-only are hidden" is wrong:** it is 64 at most (see plan 102's review).
4. **Open question 2 should be the recommendation.** Skills reach only plugin-loading clients. MCP prompts on the server reach Claude, ChatGPT and the dashboard assistant (plan grover-admin 102) alike. Ship `customer-questions-report` as a server prompt first and keep the skill as a thin wrapper.
5. **Evals send real tool calls.** Running them weekly against dev from this public repo's CI needs a dev partner credential stored as a CI secret. That is a new long-lived secret for a public repo, and Will should approve it explicitly.
