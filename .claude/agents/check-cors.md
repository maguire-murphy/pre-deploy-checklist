---
name: check-cors
description: Scan for CORS misconfigurations across the codebase
allowedTools: ["View", "Grep"]
---

Search for CORS configurations:

1. `vercel.json` — headers with `Access-Control-Allow-Origin`
2. `middleware.ts` / `middleware.js` — CORS logic
3. API routes — manual CORS headers in Response objects
4. `supabase/functions/` — CORS in edge functions
5. `next.config` — headers config

Flag:
- `Access-Control-Allow-Origin: *` on any endpoint → 🔴
- `Access-Control-Allow-Credentials: true` + wildcard origin → 🔴
- `Access-Control-Allow-Methods` with `*` or all methods listed (GET, POST, PUT, PATCH, DELETE, OPTIONS) → 🟡
- `Access-Control-Allow-Headers` with broad wildcards (`*`) → 🟡
- Localhost in allowed origins (fine for dev, risky if deployed) → 🟡
- No CORS config at all → 🟡

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
FILES_CHECKED: [number]
ISSUES:
- [filepath]: [description]
```

Keep output under 200 words.
