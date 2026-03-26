[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-blueviolet.svg)](https://docs.anthropic.com/en/docs/claude-code/overview)
[![Checks](https://img.shields.io/badge/checks-12-green.svg)](#the-12-checks)

# Pre-Deployment Checklist when Building with Claude Code

### Security and reliability checks for Supabase + Vercel + Next.js apps that are powered by Claude Code subagents.

A lot of new people are attempting to build and ship with Claude Code, it is now quick and easy to build something worth using but if you don't secure your applications you are putting yourself at risk. I wanted to share the best way to secure the stack that I use the most.
 
So I built this. It's a pre-deployment plug-in that runs entirely inside [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview), using subagents to audit your app in parallel without eating your main session's context window. It checks twelve things every app should have locked down before launch, and it can auto-fix most of what it finds. 

It's built for the stack that most builders are shipping with right now: **Supabase + Vercel + Next.js**. That's intentional — the [Supabase MCP](https://supabase.com/docs/guides/getting-started/mcp) integration lets the agents query your database schema, check RLS policies, review auth configuration, and generate fix SQL directly, all from the terminal.
 
```
/pre-deploy
```
 
That's it. Twelve checks run in parallel, each in its own context window, and you get back a single pass/fail report with a **SHIP IT** or **DO NOT SHIP** verdict. If issues are found, it offers to generate and apply fixes with your approval.
 
---
 
## Why I Built This
 
I'm building and shipping apps fast, and so is everyone around me. The gap isn't in features — it's in the boring stuff that sits between "it works" and "it's ready for real users." Things like:
 
- Does every table have Row-Level Security?
- Are your API routes validating input?
- Is your CORS config open to the entire internet?
- Do your error pages show raw stack traces?
- Are your most common queries indexed?
- Can you actually roll back if a deploy goes bad?
 
If you don't understand what to do these checks take 30-60 minutes to do manually across a codebase. Most people skip them. This automates the whole process in under two minutes using Claude Code's subagent architecture. Now, every check runs in isolation, uses only the tools it needs, and returns a short summary. Your main session stays clean.
 
---

## Setup
 
### Prerequisites
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) installed (`npm install -g @anthropic-ai/claude-code`)
- [Supabase MCP](https://supabase.com/docs/guides/getting-started/mcp) connected to a **dev** project (never production)
- [Vercel CLI](https://vercel.com/docs/cli) installed and authenticated
 
### Install

From inside Claude Code:
```
/plugin marketplace add maguire-murphy/pre-deploy-checklist
/plugin install pre-deploy-checklist@pre-deploy-checklist
```

Then connect Supabase MCP to your dev project (never production):
```bash
claude mcp add supabase --url "https://mcp.supabase.com/mcp?project_ref=YOUR_PROJECT_REF"
```

Replace `YOUR_PROJECT_REF` with your project ID from Supabase dashboard → Project Settings → General.

### Save Tokens (Recommended)

Route subagents to Sonnet instead of Opus. Add to your shell profile to persist:
```bash
# macOS / Linux — add to .zshrc or .bashrc
export CLAUDE_CODE_SUBAGENT_MODEL="claude-sonnet-4-6"

# Windows PowerShell — add to $PROFILE
$env:CLAUDE_CODE_SUBAGENT_MODEL="claude-sonnet-4-6"
```
 
### Run
 
```
/pre-deploy
```
 
---
 
## How It Works
 
```
You type /pre-deploy
        │
        ▼
   Main Agent (orchestrator — stays lightweight)
        │
        ├───── 12 check agents spawn in parallel ─────┐
        │                                              │
        │  check-rls          check-validation         │
        │  check-cors         check-rate-limiting      │
        │  check-auth-config  check-error-handling     │
        │  check-indexes      check-logging            │
        │  check-alerts       check-rollback           │
        │  check-dependencies check-env                │
        │                                              │
        │  Each gets:                                  │
        │  · Fresh context window                      │
        │  · Restricted tool access                    │
        │  · 200-300 word output cap                   │
        │  · Runs on Sonnet (not Opus) for cost        │
        │                                              │
        ├──────────────────────────────────────────────┘
        │
        ▼
   Compiled report → SHIP IT / DO NOT SHIP
        │
        ▼ (if issues found)
   Fix agents spawn for affected domains
        │
   ┌────┼────┐
   ▼    ▼    ▼
  fix-  fix-  fix-
  db    code  infra
        │
        ▼
   Changes presented → you approve → applied
```
 
Every subagent is defined as a markdown file in `agents/`. No SDK, no build step. The orchestrator fans them out, collects results, and compiles the report. If there are findings, it spawns fixer agents only for the domains that need attention.
 
---
 
## The 12 Checks
 
| Command | What It Checks |
|---------|---------------|
| `/check-rls` | RLS enabled on all tables, policies scope to `auth.uid()`, no `service_role` in client code |
| `/check-validation` | Zod validation on API routes and server actions, safe `.rpc()` calls |
| `/check-cors` | No wildcard origins, no credential + wildcard combos, consistent config |
| `/check-rate-limiting` | Rate limit library installed, middleware configured, public endpoints protected |
| `/check-auth-config` | Token TTLs ≤ 30 min, email confirmation, MFA settings, password requirements |
| `/check-error-handling` | `error.tsx`/`loading.tsx`/`not-found.tsx` exist, Supabase errors handled, try/catch coverage |
| `/check-indexes` | Foreign key indexes, query patterns cross-referenced with existing indexes, scan stats |
| `/check-logging` | Structured logger installed, no `console.log` in prod, no PII in logs, no empty catches |
| `/check-alerts` | Monitoring SDK installed and configured, error boundaries report to it |
| `/check-rollback` | Vercel CLI authenticated, recent deployments exist, down migrations present, git tags |
| `/check-dependencies` | `npm audit` vulnerabilities, outdated core packages |
| `/check-env` | `.env` not in git, `.gitignore` coverage, no hardcoded secrets, `NEXT_PUBLIC_` exposure |
 
Run any check individually or all twelve at once with `/pre-deploy`.
 
---
 
## Fixers
 
When checks find issues, three fixer agents can generate and apply fixes:
 
| Command | What It Fixes |
|---------|--------------|
| `/fix-database` | RLS policies, indexes, auth config — generates SQL and waits for your approval |
| `/fix-codebase` | Zod schemas, CORS config, error boundaries, logging, rate limiting middleware |
| `/fix-infra` | Auth settings via MCP, monitoring boilerplate, migration templates, `.env.example` |
 
Fixers work standalone (they do a quick assessment first) or after `/pre-deploy` (they receive findings and skip straight to fixes). All changes require explicit approval before applying.
 
---
 
## Example Output
 
```
| #  | Check          | Status | Key Findings                                    |
|----|----------------|--------|-------------------------------------------------|
| 1  | RLS            | 🔴     | `profiles` table has RLS disabled                |
| 2  | Validation     | 🟡     | 2 POST routes missing Zod validation             |
| 3  | CORS           | 🟢     | Explicit origin allowlist configured              |
| 4  | Rate Limiting  | 🟡     | No rate limiting on /api/auth/* routes            |
| 5  | Auth Config    | 🟢     | All settings follow best practices                |
| 6  | Error Handling | 🟢     | error.tsx exists in all route groups               |
| 7  | Indexes        | 🟡     | Missing index on orders.user_id foreign key       |
| 8  | Logging        | 🟡     | 12 console.log calls in production code           |
| 9  | Alerts         | 🔴     | No monitoring SDK detected                        |
| 10 | Rollback       | 🟢     | Vercel CLI authenticated, 5 recent deployments    |
| 11 | Dependencies   | 🟡     | 2 moderate vulnerabilities in transitive deps     |
| 12 | Environment    | 🟢     | .env.example exists, no secrets in git            |
 
Result: DO NOT SHIP — 2 blockers found
 
Would you like me to generate fixes for these issues?
```
 
---
 
## What's In The Box
 
```
pre-deploy-checklist/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata
│   └── marketplace.json     # Marketplace listing
│
├── commands/                # 17 slash commands — type /command-name to run
│   ├── pre-deploy.md        # Orchestrator: all 12 checks + fix phase
│   ├── check-*.md           # Individual check shortcuts (12 files)
│   ├── fix-*.md             # Fixer shortcuts (3 files)
│   └── security-review.md
│
├── agents/                  # 16 subagent definitions — the actual logic
│   ├── check-*.md           # Check agents with scoped tools + output caps
│   ├── fix-*.md             # Fixer agents with approval gates
│   └── security-reviewer.md
│
├── skills/                  # 4 skills — enforce patterns during normal coding
│   ├── validation/SKILL.md
│   ├── error-handling/SKILL.md
│   ├── logging/SKILL.md
│   └── security/SKILL.md
│
├── hooks/
│   └── hooks.json           # 5 hooks — real-time guardrails on every file write
│
├── .mcp.json
├── README.md
└── .gitignore
```
 
**Commands** are what you type. **Agents** are the isolated workers that do the analysis. **Skills** teach Claude your patterns while you're coding (not during audits). **Hooks** fire automatically on every file write to catch issues in real-time — missing Zod, wildcard CORS, exposed service role keys.
 
---
 
## Token Conservation
 
This was designed to minimize cost. Running twelve audit agents sounds expensive — it doesn't have to be.
 
1. **Subagent model routing** — Subagents run on Sonnet, your main session stays on Opus. Set `CLAUDE_CODE_SUBAGENT_MODEL="claude-sonnet-4-6"` in your shell profile.
2. **Single parallel batch** — All 12 agents spawn simultaneously. No sequential turns burning orchestrator tokens.
3. **Scoped tool access** — Each agent only gets the tools it needs. `check-cors` gets Grep and View. `check-rls` gets Supabase MCP. No wasted calls.
4. **Output caps** — Every agent returns under 300 words. The orchestrator sees ~3,000 words total, not the thousands of lines the agents actually processed.
5. **Smart fix spawning** — Fixers only spawn for domains with findings. All-green domains skip entirely.
6. **Compact skills** — Under 10 lines each. They load into context when Claude writes matching files but don't bloat anything else.

## Troubleshooting
 
| Problem | Fix |
|---------|-----|
| MCP connection failed | Verify project ref. Run `claude mcp list` to confirm connection. |
| Vercel CLI not found | `npm i -g vercel` then `vercel login` |
| Agent returned empty | Check your project uses the expected structure (`app/api/`, `supabase/`, etc.) |
| Checks running slow | Set `CLAUDE_CODE_SUBAGENT_MODEL` to Sonnet — faster for focused analysis |
| Hook not firing | Run `claude settings` to verify hooks loaded |
| Fix agent did nothing | Fixers present changes and wait for approval — check the prompt |
 
---
 
## Customizing
 
**Add a check:** Create a `.md` in `agents/` with YAML frontmatter (`allowedTools`, output format) and a matching `.md` in `commands/` as the slash shortcut. Add it to the agent list in `commands/pre-deploy.md`.

**Modify a check:** Edit the agent's `.md` file directly. Plain markdown, no build step.

**Add a skill:** Create a folder in `skills/` with a `SKILL.md` file containing `globs` targeting the files it should apply to.

**Add a hook:** Edit `hooks/hooks.json` — bash one-liners on `PostToolUse` or `PreToolUse`.
 
--- 
  
## Contributing
 
This is opinionated toward Supabase + Vercel + Next.js. The subagent patterns work with any stack — swap out Supabase MCP for your database and Vercel CLI for your host. PRs welcome.
 
## Credits
 
Inspired by the pre-deployment checklist blog post from Arjay McCandless
 
## License
 
MIT
