Run the pre-deployment checklist by spawning subagents for each check. This keeps the main context clean.

## Phase 1 — Audit

Spawn ALL of the following subagents **in parallel** using the Agent tool. Each agent is defined in `.claude/agents/` and will return a short structured summary. There are no dependencies between agents — they each run in isolated contexts.

- `check-rls` — RLS audit on all public tables (includes write policies + service_role scan)
- `check-indexes` — Database index coverage
- `check-auth-config` — Auth settings review (includes MFA + password policy)
- `check-validation` — Input validation on API routes and server actions
- `check-cors` — CORS configuration scan (includes methods/headers)
- `check-rate-limiting` — Rate limiting coverage
- `check-error-handling` — Error boundaries and Supabase error handling
- `check-logging` — Logging standards, sensitive data, and PII
- `check-alerts` — Error monitoring integration
- `check-rollback` — Rollback readiness
- `check-dependencies` — Dependency vulnerabilities and outdated packages
- `check-env` — Environment variable security

After all 12 agents return, compile their results into a single summary table:

| # | Check | Status | Key Findings |
|---|-------|--------|-------------|
| 1 | Authorization (RLS) | 🟢/🟡/🔴 | [from agent] |
| 2 | Input Validation | 🟢/🟡/🔴 | [from agent] |
| 3 | CORS | 🟢/🟡/🔴 | [from agent] |
| 4 | Rate Limiting | 🟢/🟡/🔴 | [from agent] |
| 5 | Auth Config | 🟢/🟡/🔴 | [from agent] |
| 6 | Error Handling | 🟢/🟡/🔴 | [from agent] |
| 7 | Database Indexes | 🟢/🟡/🔴 | [from agent] |
| 8 | Logging | 🟢/🟡/🔴 | [from agent] |
| 9 | Alerts | 🟢/🟡/🔴 | [from agent] |
| 10 | Rollback | 🟢/🟡/🔴 | [from agent] |
| 11 | Dependencies | 🟢/🟡/🔴 | [from agent] |
| 12 | Environment | 🟢/🟡/🔴 | [from agent] |

Count the statuses:
- If any 🔴: **DO NOT SHIP** — list blockers
- If only 🟡 and 🟢: **SHIP WITH CAUTION** — list warnings
- If all 🟢: **SHIP IT**

Do NOT re-run any checks yourself. Only compile the agent results.

## Phase 2 — Fix Planning

If any 🔴 or 🟡 results exist, enter the fix phase:

1. Group all findings by domain:
   - **Database:** RLS, indexes, auth config findings → `fix-database`
   - **Codebase:** Validation, CORS, error handling, logging, rate limiting findings → `fix-codebase`
   - **Infrastructure:** Alerts, rollback, dependencies, env findings → `fix-infra`

2. Ask the user: **"Would you like me to generate fixes for these issues?"**

3. If the user says yes, spawn only the fixer agents that have findings (skip fixers for domains where all checks passed green). Pass each fixer a summary of its relevant findings. Spawn applicable fixers in parallel:
   - `fix-database` — with RLS, index, and auth config findings
   - `fix-codebase` — with validation, CORS, error handling, logging, and rate limiting findings
   - `fix-infra` — with alerts, rollback, dependency, and env findings

4. Compile the fix proposals from all fixers into a combined numbered list, organized by priority (🔴 fixes first, then 🟡).

5. Present the combined fix plan to the user and ask: **"Which fixes would you like me to apply? (all / specific numbers / none)"**

6. Apply only the approved fixes by instructing the relevant fixer agents.

7. If a fixer agent fails or errors, skip those fixes and note the failure in the final report.

If all checks passed 🟢, skip the fix phase entirely.

## Token Conservation

Set `CLAUDE_CODE_SUBAGENT_MODEL` to `claude-sonnet-4-6` in your environment before running. All subagents are scoped to use Sonnet — they handle focused analysis tasks that don't need Opus-level reasoning.
