---
name: check-rate-limiting
description: Check API routes for rate limiting coverage
allowedTools: ["View", "Grep"]
---

Check rate limiting coverage:

1. Is a rate limiting library installed? Search `package.json` for `@upstash/ratelimit`, `rate-limiter-flexible`, or similar
2. Check `middleware.ts` for global rate limiting
3. For each API route in `app/api/`:
   - Is rate limiting applied?
   - Is it a public endpoint (no auth check)?
   - Is it a mutation (POST/PUT/DELETE)?

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
LIBRARY: [name or "none"]
ROUTES:
- [route]: [public|auth] [mutation|read] [rate-limited: yes|no]
UNPROTECTED_COUNT: [number]
```

Rules:
- 🔴 if public mutation endpoints have no rate limiting
- 🟡 if auth-protected routes lack rate limiting
- 🟢 if all public endpoints are rate limited
- Keep output under 200 words
