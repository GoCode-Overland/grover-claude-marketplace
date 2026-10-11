---
name: usage-dashboard
description: Grover staff only. Grover's internal company dashboard from the Grover MCP server's get_usage_dashboard tool - signups, MAU/WAU, per-company ARR, at-risk and up-renewal, green/amber/red company health with reasons, and platform health. Use when a Grover founder or staff member asks "how is Grover doing", "how is <company> doing", "is anything broken", or for usage, ARR, renewal or health numbers.
---

# Grover company dashboard (Grover staff only)

The output names Grover's customers and carries their billing figures. Keep it in
this conversation: never paste a customer name or a figure into a commit, issue,
public channel or document unless the person asking says to.

## Call the tool

Find it with `ToolSearch` query `get_usage_dashboard`. It is on the Grover MCP
server: `mcp__plugin_grover_grover__get_usage_dashboard` with this plugin, or
`mcp__claude_ai_Grover__get_usage_dashboard` through the claude.ai Grover
connector. Either is the same tool.

- "How is Grover doing": `{}` (every section).
- One company: `{ "company": "<id, subdomain, domain or exact name>" }`.
- Part of it: `{ "sections": [...] }` with any of `usage`, `revenue`,
  `company_health`, `platform_health`.
- `windowDays` (default 30) sets the comparison window; `year` gives an annual
  view of acquisition and revenue; `includeStripe: false` skips the live Stripe
  price read if it is slow.

If the tool answers with a forbidden-role error, the account is not Grover
staff with all-company access (`grover_owner` or `grover_admin`, `allCompanies`). Say exactly that, and point to the staff
web page (admin.getgrover.ai, "How Grover is doing") instead of guessing numbers.
If the tool is missing, the Grover server is not connected: run `/mcp`.

## Render: "how is Grover doing" on one screen

Call the tool once with no arguments (all four sections). Then render, numbers
before adjectives, in this order and nothing more:

1. **First line**: generated time, window.
2. **KPI row** (from `headline`), one line or a 6-cell table: signups all-time
   (MTD), MAU (vs `mauPrev`), WAU, ARR, projected ARR, paying companies.
   Money is in cents: divide by 100.
3. **Company health table** (`companyHealth.companies`, already red-first):
   status, company, ARR, MAU (delta vs previous), reasons. Copy each reason as
   `code why` verbatim (`R4 subscription past_due`); never invent a reason or
   a score. Show every red and amber row; collapse greens into a count unless
   asked. Notes (renewal soon, override) go in a last column, short.
4. **At risk** (`atRisk`) with ARR and reasons, and **up-renewal**
   (`upRenewal`) with the ratio. Skip a list that is empty, saying so in one line.
5. **Platform health** (`platformHealth`): overall status, then one line per
   section that is not green, with its reasons. One line "all other sections
   green" for the rest.
6. **Not measured**: one line listing `platformHealth.notMeasured` briefly, so
   nobody reads silence as health.
7. Caveats that apply to THIS run only: Stripe reads failed (`stripe.failed`),
   MCP history starts on `companyHealth.mcpHistoryStartsOn`.

Small numbers: lead with the people affected, not percentages.

**Drill-down**: "how is <company> doing" → call with `{ "company": "<id, subdomain,
domain or exact name>" }`. It returns the company's health with reasons, usage
row, admins with last sign-in, failed/stuck knowledge docs, push errors and home
circles. `error: "ambiguous"` lists matches: ask which one.

**Sections only**: `{ "sections": ["platform_health"] }` for "is anything broken",
`["company_health"]` for "which customers need attention", `["usage", "revenue"]`
for the #972 view.

Rules (thresholds, reason codes) are returned in the response (`rules`,
`companyHealth.rules`, each platform section's `rule`): quote them when asked
"why is X amber", don't paraphrase from memory.
