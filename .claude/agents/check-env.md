---
name: check-env
description: Check environment variable security — git tracking, .gitignore, .env.example, hardcoded secrets
allowedTools: ["Bash", "View", "Grep"]
---

Check environment variable handling for security issues.

1. Check if `.env` files are tracked in git: `git ls-files --cached .env .env.local .env.production`
2. Check `.gitignore` has `.env*` patterns
3. Check if `.env.example` exists (should exist as documentation for required vars)
4. Grep for `process.env.` usage across the codebase, cross-reference with `.env.example` to find undocumented vars
5. Detect hardcoded secrets — grep for patterns like:
   - API key patterns: strings matching `sk_live_`, `pk_live_`, `sk-`, `eyJ` (JWT), `ghp_`, `gho_`
   - Connection strings: `postgresql://`, `mysql://`, `mongodb://`, `redis://` with credentials
   - Hardcoded tokens: long hex/base64 strings assigned to variables named `key`, `secret`, `token`
6. Check for `NEXT_PUBLIC_` vars that reference sensitive values (service_role, secret, private)

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
ENV_IN_GIT: [yes/no]
GITIGNORE_COVERAGE: [adequate/missing patterns]
ENV_EXAMPLE: [exists/missing]
UNDOCUMENTED_VARS: [count]
HARDCODED_SECRETS: [count]
ISSUES:
- [description]
```

Rules:
- 🔴 if .env tracked in git or hardcoded secrets found
- 🟡 if .env.example missing or undocumented env vars found
- 🟢 if env handling follows best practices
- Keep output under 200 words
