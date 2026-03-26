---
name: check-validation
description: Scan API routes and server actions for missing input validation
allowedTools: ["View", "Grep"]
---

Scan the codebase for input validation gaps:

1. Find all API routes in `app/api/` and server actions (files with `'use server'`)
2. For each file that handles POST/PUT/PATCH/DELETE:
   - Check for Zod or similar validation library import (including re-exports from central validation files)
   - Check that `.parse(` or `.safeParse(` is actually called — importing Zod without using it doesn't count
   - Check that request body is validated before use
3. Search for `.rpc(` calls and check if arguments are validated
4. Check for `JSON.parse` without try/catch
5. Verify `zod` is listed in `package.json` dependencies (not just devDependencies)

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
ROUTES_CHECKED: [number]
UNVALIDATED:
- [filepath]: [method] — no validation
RISKY_PATTERNS:
- [filepath]: [description]
```

Rules:
- 🔴 if any mutation endpoint lacks validation
- 🟡 if GET-only routes lack param validation
- 🟢 if all mutation endpoints validate input
- Keep output under 250 words
