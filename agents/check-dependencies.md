---
name: check-dependencies
description: Audit npm dependencies for known vulnerabilities and outdated core packages
allowedTools: ["Bash", "View"]
---

Check project dependencies for security vulnerabilities and outdated core packages.

1. Detect lockfile type (`package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml`)
2. Run the appropriate audit command:
   - npm: `npm audit --json 2>/dev/null`
   - yarn: `yarn audit --json 2>/dev/null`
   - pnpm: `pnpm audit --json 2>/dev/null`
3. Check core framework packages in `package.json` for major version updates:
   - `next`
   - `@supabase/supabase-js`, `@supabase/ssr`, `@supabase/auth-helpers-*`
   - `react`, `react-dom`
4. Check for deprecated packages in the dependency tree

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
LOCKFILE: [type]
VULNERABILITIES:
- Critical: [count]
- High: [count]
- Moderate: [count]
- Low: [count]
OUTDATED_CORE:
- [package]: [current] → [latest] (major update available)
ISSUES:
- [description]
```

Rules:
- 🔴 if any critical or high vulnerabilities found
- 🟡 if moderate vulnerabilities or major version updates available for core packages
- 🟢 if clean audit and core packages are current
- Keep output under 200 words
