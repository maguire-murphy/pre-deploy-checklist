---
name: check-rollback
description: Verify rollback readiness for Vercel deployments and Supabase migrations
allowedTools: ["Bash", "View"]
---

Check rollback readiness:

1. Run `vercel whoami` — is CLI authenticated?
2. Run `vercel ls --prod --limit 3` — are there recent deployments to roll back to?
3. Check for Supabase migration files in `supabase/migrations/`
4. Check if down migrations exist alongside up migrations
5. Check if git tags exist for recent deployments: `git tag --sort=-creatordate | head -5`

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
VERCEL_CLI: [authenticated|not found]
RECENT_DEPLOYMENTS: [count]
MIGRATION_FILES: [count]
DOWN_MIGRATIONS: [count]
GIT_TAGS: [list or "none"]
ISSUES:
- [description]
```

Rules:
- 🔴 if Vercel CLI not authenticated or no deployments exist
- 🟡 if no down migrations or no git tags
- 🟢 if CLI ready, deployments exist, and rollback path is clear
- Keep output under 150 words
